---
description: "Understand lock contention, leases, renewals, fencing tokens, and handling after loss of ownership through the Redis multi-process paradigm."
icon: lock
---

# Distributed Locks

Zongsoft.Services.Distributing is used to coordinate short-term access to the same resource by multiple processes. Discussions currently mainly maintains the consistency of topics, posts and statistics through [database transaction](../../data/transactions.md), without directly using distributed lock; this page uses the existing Redis multi-process examples and tests in the framework.

## First Distinguish Three Concepts

* **mutually exclusive**: There is only one holder of the same lock key within the validity period. The lock key must correspond to a real shared resource, and different processes cannot be mutually exclusive if they use different keys.
* **lease**: The effective duration of the lock on the server. The server can automatically expire the lock after the process exits, but the end of the lease will not interrupt the code already running in the old process.
* **fencing token**: The incremental number generated each time the lock is successfully acquired. The protected store records the maximum number that has been accepted and rejects late writes with smaller numbers, thereby restricting the old process that lost the lock from continuing to modify the resource.

💡 Locks coordinate executors; transactions protect a group of writes within a database. When you need to ensure that multiple tables are updated successfully at the same time, you should still use transactions; when you need to avoid repeated submissions, you still need idempotent constraints.

## Core Types

| Type | What you care about when using |
| --- | --- |
| [IDistributedLockManager](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/Distributing/IDistributedLockManager.cs) | AcquireAsync gets the handle, GetExpiryAsync queries the remaining lease, and ReleaseAsync releases by ownership token. |
| [IDistributedLock](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/Distributing/IDistributedLock.cs) | EnterAsync waits for entry, RenewAsync manually renews, and releases the handle to end use. |
| [DistributedLockOptions](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/Distributing/DistributedLockOptions.cs) | Expiry must be positive; RenewalInterval specifies the auto-renewal interval. |
| [DistributedLockBase](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/Distributing/DistributedLockBase.cs) | Encapsulates waiting, holding time, renewal and release; specific atomic operations are the responsibility of the implementation. |
| [IDistributedLockTokenizer](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Services/Distributing/IDistributedLockTokenizer.cs) | Generate a byte ownership token Token; this random token has a different purpose than the incremented FencingToken. |

## Lock Object Status

| Properties | meaning |
| --- | --- |
| Key / Token | The protected resource key, and the ownership token obtained this time. |
| IsHeld / IsUnheld | Whether the handle is recorded as held; IsHeld itself does not check whether the lease is exceeded. |
| IsExpired | Determine whether it has expired based on the holding time of the local record. |
| IsLocked / IsUnlocked | Whether held and unexpired, and the opposite status. |
| FencingToken | The fencing token successfully obtained this time; if Redis fails to obtain it, it will be zero. Other implementations that do not support this capability can also return zero. |

These states are not a real-time, strongly consistent lookup of server-side ownership. After process suspension, network delay, or server state changes, local judgment cannot replace the verification of protected storage.

## Basic Usage

The Redis slave example competes for the same lock in a loop, selects normal lease or automatic renewal according to configuration, and combines command cancellation with the total timeout. Here is the original snippet of the loop acquiring the lock and waiting for entry:

Source: [framework/externals/redis/samples/distributedlock/slaver/RunCommand.cs](https://github.com/Zongsoft/framework/blob/main/externals/redis/samples/distributedlock/slaver/RunCommand.cs#L24) (excerpt; see source for context).

{% code title="RunCommand.cs" %}
```csharp
await using var locker = settings.RenewalInterval.HasValue ?
	await redis.AcquireAsync(Utility.Keys.Lock, new DistributedLockOptions(settings.Expiry) { RenewalInterval = settings.RenewalInterval }, linked.Token) :
	await redis.AcquireAsync(Utility.Keys.Lock, settings.Expiry, linked.Token);
await locker.EnterAsync(linked.Token);
```
{% endcode %}

settings, redis and linked are established by the same OnExecuteAsync method. The scope of await using is a loop; the lock is released when the loop completes or an exception is thrown. EnterAsync queries the remaining lease if it is not yet held and waits before trying to acquire it; the caller should pass in a cancellation token for the wait.

If you are not ready to wait, you should check whether the handle you just obtained is IsLocked, and then decide whether to start working. Redis' current acquisition failure will return an unheld handle; just checking that it is not empty is not enough to prove that the lock has been grabbed. The framework tests clearly verify this difference:

Source: [framework/externals/redis/test/RedisDistributedLockTests.cs](https://github.com/Zongsoft/framework/blob/main/externals/redis/test/RedisDistributedLockTests.cs#L40) (excerpt; see source for context).

{% code title="RedisDistributedLockTests.cs" %}
```csharp
await using var first = await cache.AcquireAsync(key, TimeSpan.FromSeconds(2));
Assert.True(first.IsLocked);
Assert.True(first.FencingToken > 0);

await using var rejected = await cache.AcquireAsync(key, TimeSpan.FromSeconds(2));
Assert.True(rejected.IsUnheld);
Assert.Equal(0, rejected.FencingToken);
```
{% endcode %}

The snippet comes from SuccessfulAcquisitions_ReturnStrictlyIncreasingFencingTokens. The complete test also releases the first lock, reacquires it, and verifies that the fencing token increases. cache and key are from the test fixture, not Discussions configuration.

## Renewal and Loss of Ownership

Redis does not automatically renew by default. When enabled, RenewalInterval must be greater than zero and less than Expiry; a null or zero value indicates off, and negative values are rejected. Actual test demonstrates renewal with 300ms lease, 75ms interval:

Source: [framework/externals/redis/test/RedisDistributedLockTests.cs](https://github.com/Zongsoft/framework/blob/main/externals/redis/test/RedisDistributedLockTests.cs#L111) (excerpt; see source for context).

{% code title="RedisDistributedLockTests.cs" %}
```csharp
var options = new DistributedLockOptions(TimeSpan.FromMilliseconds(300))
{
	RenewalInterval = TimeSpan.FromMilliseconds(75),
};
await using var renewed = await cache.AcquireAsync("automatic", options);
```
{% endcode %}

These shorter durations are for testing, production environments should be set up based on scheduling latency and network round-trips. Redis starts the renewal loop after the first acquisition is successful or the EnterAsync competition is successful; it stops when the handle is released and waits for the end of the loop.

A successful manual RenewAsync resets the local holding time; the caller should stop the protected operation when false is returned. The automatic renewal cycle will end if it encounters a failure or exception. The exception path will be logged and the holding status will be cleared. When an exception is thrown by a manual call, it should also be treated as if the ownership is uncertain, and writing cannot be continued just because the local state is still held temporarily.

{% hint style="warning" %}
🚨 The current base class's EnterAsync only tries to get when IsUnheld. An old handle that has expired but is still recorded as IsHeld will not automatically get a new lease by EnterAsync again. The old scope should be ended and the handle reacquired. Renewal will not automatically cancel business work that has already been started.
{% endhint %}

## Redis Implementation

[RedisService.DistributedLock.cs](https://github.com/Zongsoft/framework/blob/main/externals/redis/src/RedisService.DistributedLock.cs) first uses SET NX with validity period to compete for the lock key. After success, the counter corresponding to the same key suffix: FENCE is incremented. The loser gets a zero number; an exception occurs when allocating a number and attempts to release the lock by ownership token.

Release and renewal respectively execute Lua scripts that compare Token and then delete it, and compare Token and then extend TTL. Therefore, the old process cannot delete the lock held by the new process simply by virtue of the same Key. All keys are also prefixed with RedisService.Namespace.

{% hint style="warning" %}
🚨 The fence number depends on the continuity of the corresponding Redis counter. Do not clean up lock prefixes or :FENCE counters while there are still executors working, and do not understand this single Redis implementation as a consensus service under cross-cluster failures.
{% endhint %}

## How Does Fencing Token Protect Storage?

The multi-process example submits FencingToken once each after entering the critical section and after the simulation work is completed. The expiry scenario is deliberately made to work longer than the lease, allowing the reader to observe late writes from the old holder; mutex and renew are used for comparison.

| Example scenario | Observation target |
| --- | --- |
| mutex | The critical section is shorter than the lease, see if overlapping entries occur. |
| expiry | Critical section exceeds lease, observe Violations and Stale counts. |
| renew | The longer critical section cooperates with automatic renewal and observes whether the holding status is maintained during renewal. |

See [Redis distributed lock example](https://github.com/Zongsoft/framework/blob/main/externals/redis/samples/distributedlock/README.zh-Hans.md) for complete operation. It contains two real projects, master and slave, use independent running identities, and support dedicated Redis connections.

The example WriteFenceAsync demonstrates token comparison using a read followed by a write. The protected storage must perform “checking the maximum token” and “committing data and updating the maximum token” as one atomic operation, such as database condition update or server-side script; directly copying separate reads and writes will leave a concurrency window. A single example result cannot prove security under arbitrary pauses and network failures.

## Get the Redis Lock Manager

The plugin registers a named lock manager provider through RedisServiceProvider. The real caller is WeChat CredentialManager, which parses the provider from the application service and obtains the service according to the cache configuration name:

Source: [framework/externals/wechat/src/CredentialManager.cs](https://github.com/Zongsoft/framework/blob/main/externals/wechat/src/CredentialManager.cs#L76) (excerpt; see source for context).

{% code title="CredentialManager.cs" %}
```csharp
public static Services.Distributing.IDistributedLockManager Locker
{
	get => _locker ??= ApplicationContext.Current.Services.Resolve<IServiceProvider<Services.Distributing.IDistributedLockManager>>()?.GetService(GetCacheName());
	set => _locker = value;
}
```
{% endcode %}

The configuration entry is /Externals/Redis/ConnectionSettings. For the ownership and reuse of the naming service, see [service resolution](locating.md). For plugin deployment, see [Redis](../../externals/projects/redis.md). Discussions does not have a ready-made Redis lock connection, so its database name cannot be directly copied as the Redis connection name.

## Real Scene

CredentialManager.GetCredentialAsync obtains a short lease for the credential refresh request after neither the local cache nor the shared cache has valid credentials. Its current implementation only checks that locker is not empty, without checking IsLocked or calling EnterAsync. Combined with the behavior of Redis returning unheld handles, this existing business implementation cannot be used as a complete example that guarantees mutual exclusion; the holding state processing should be checked and completed during integration. This article uses Redis slaver and testing as the main examples of acquisition, waiting, and renewal.

## Usage Suggestions

* Keep the critical section as short as possible, and remove user interaction, unbounded waiting and batch tasks from the lock scope.
* Pass in the cancellation token and set the waiting limit and lease time respectively.
* Design compensation and idempotent handling for possible retries when lock acquisition, renewal, or a business write has an uncertain outcome.
* Use scope to release handles; asynchronous call chains prefer await using.
* When late writes need to be rejected, the fencing token verification must be implemented in the final storage and cannot be compared only in the application process.

## Related Resources

* [Service Resolution and Ownership](locating.md)
* [Redis project](../../externals/projects/redis.md)
* [Transactions and Consistency](../../data/transactions.md)
* [Redis lock integration test](https://github.com/Zongsoft/framework/blob/main/externals/redis/test/RedisDistributedLockTests.cs)
