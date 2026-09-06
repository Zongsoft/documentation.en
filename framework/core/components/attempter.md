---
description: "Zongsoft.Components Attempter Failed attempt statistics and lock window."
icon: shield-halved
---

# Attempter

`Attempter` is used to count the number of failed attempts for a certain key and enter the locking period after reaching the limit. It is often used in security scenarios such as login, verification code, and secondary confirmation of sensitive operations.

The attempt tracker maintains the "number of failures" and "lock window" in the cache. The caller only needs to check before verification, register after failure, and clear after success.

## Key Types

| Type | Description |
| --- | --- |
| `IAttempter` | The attempt tracker interface defines checking, successful completion and failure registration. |
| `Attempter` | The default implementation is based on distributed cache and `ISequence` incremental counting of failures. |
| `IAttempterOptions` | Try limiting the options interface. |
| `AttempterOptions` | Default attempts to restrict options. |

## Working Mechanism

{% stepper %}
{% step %}
## Check

Call `CheckAsync(key)` to check if the current key still allows attempts.
{% endstep %}

{% step %}
## Fail

Call `FailAsync(key)` on failure. Implementations will increment the number of failures via the cached `ISequence` capability.
{% endstep %}

{% step %}
## Window / Period

Set a short window when the limit is not reached; set a locking period when the limit is reached for the first time.
{% endstep %}

{% step %}
## Done

After success, call `DoneAsync(key)` to clear the cache key to avoid the number of historical failures from continuing to affect the user.
{% endstep %}
{% endstepper %}

Source: [framework/Zongsoft.Core/src/Security/Privileges/Authenticators.Identity.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/Authenticators.Identity.cs#L86) (excerpt; see source for context).

{% code title="Authenticators.Identity.cs" %}
```csharp
public async ValueTask<Ticket> VerifyAsync(string key, Requirement requirement, string scenario, Parameters parameters, CancellationToken cancellation = default)
{
	if(string.IsNullOrWhiteSpace(requirement.Identity))
	{
		if(string.IsNullOrEmpty(key))
			throw new AuthenticationException(SecurityReasons.InvalidIdentity, "Missing identity.");

		requirement.Identity = key;
	}

	//获取验证失败的解决器
	var attempter = this.Attempter;
	var attempterKey = $"{this.GetType().Name}:{requirement.Identity}@{requirement.Namespace}";

	//确认验证失败是否超出限制数，如果超出则返回账号被禁用
	if(attempter != null && !await attempter.CheckAsync(attempterKey, cancellation))
		throw new AuthenticationException(SecurityReasons.AccountSuspended);

	//获取当前用户的密钥信息
	var cipher = await Authentication.Servicer.Users.Passworder.GetAsync(requirement.Identity, requirement.Namespace, cancellation);

	//如果帐户不存在则验证失败
	if(cipher == null)
		throw new AuthenticationException(SecurityReasons.InvalidIdentity);

	//执行密码验证，如果成功则返回验证成功的票证
	if(await Authentication.Servicer.Users.Passworder.VerifyAsync(requirement.Password, cipher, cancellation))
	{
		//通知验证尝试成功，即清空验证失败记录
		if(attempter != null)
			await attempter.DoneAsync(attempterKey, cancellation);

		//返回验证成功的票证
		return this.CreateTicket(cipher.Identifier, requirement);
	}

	//通知验证尝试失败
	if(attempter != null)
		await attempter.FailAsync(attempterKey, cancellation);

	//抛出验证失败异常
	throw new AuthenticationException(SecurityReasons.InvalidPassword);
}
```
{% endcode %}

Discussions uses the framework security authentication portal, behind which the module challenger supplements the site identity. The above is the reality check, successful cleanup and failed registration process of the core password authenticator. The key consists of the authenticator type, user identity and namespace. Non-existing accounts are thrown directly after failure to read the key. FailAsync here cannot be understood as covering all failure types.

The cache that `Attempter` relies on needs to support the `ISequence` increment operation, otherwise failed registration cannot be counted atomically. The security module will expose the authentication attempt tracker as a plugin builtin, making it easy to adjust restriction policies through configuration.

{% hint style="warning" %}
The attempt tracker keys are normalized to lowercase and internally prefixed. The caller should use stable, non-sensitive keys, such as username, mobile phone number or business entity identification. Do not directly use plain text passwords, verification codes or tokens as keys.
{% endhint %}

When `Limit` is less than 1, it means there is no limit on the number of failures; `Window` means how long the number of failures will be retained when the threshold is not reached, and `Period` means the locking time after the threshold is reached. In a distributed environment, it is recommended to configure a distributed cache that supports atomic increment, otherwise failure statistics on multiple nodes may be unreliable.

## Reference Implementation

* [Attempter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/Attempter.cs)
* [Zongsoft.Security.plugin](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Zongsoft.Security.plugin)
