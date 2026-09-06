---
description: "Composable conditional assertions from the security module event and log filtering instructions."
icon: circle-check
---

# Predication

Predication expresses "whether the condition is true" as an object that can be executed synchronously or asynchronously. Discussions' business conditions primarily use data engine Condition; it does not have a custom PredicationBase. Therefore, the event registration of the framework security module is used here as a real use case.

## Use Delegate for Event Description

Source: [framework/Zongsoft.Security/src/Module.Events.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Module.Events.cs#L63) (excerpt; see source for context).

{% code title="Module.Events.cs" %}
```csharp
public static readonly EventDescriptor<Privileges.AuthenticatedEventArgs> Authenticated = new(Predication.Predicate<Privileges.AuthenticatedEventArgs>(OnAuthenticated), $"{nameof(Privileges.Authentication)}.{nameof(Privileges.Authentication.Authenticated)}");
public static readonly EventDescriptor<Privileges.AuthenticatingEventArgs> Authenticating = new(Predication.Predicate<Privileges.AuthenticatingEventArgs>(OnAuthenticating), $"{nameof(Privileges.Authentication)}.{nameof(Privileges.Authentication.Authenticating)}");
```
{% endcode %}

The security module uses Predication.Predicate to wrap methods into assertions corresponding to parameter types. The event descriptor and binding logic are still the responsibility of the module, and the assertion factory does not automatically complete event registration.

Source: [framework/Zongsoft.Security/src/Module.Events.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Module.Events.cs#L114) (excerpt; see source for context).

{% code title="Module.Events.cs" %}
```csharp
private static bool OnAuthenticating(Privileges.AuthenticatingEventArgs args)
{
	Current.Meter.Authentication.Authenticating.Add(1,
		new KeyValuePair<string, object>(nameof(args.Scheme), args.Scheme),
		new KeyValuePair<string, object>(nameof(args.Scenario), args.Scenario));

	return true;
}
```
{% endcode %}

This real callback first records the authentication indicator and then returns true. It shows that assertions may contain side effects; combination and short-circuiting will affect whether subsequent callbacks are executed, so assertions with side effects cannot be unconditionally rearranged as pure functions.

## Types and Combinations

| Type | responsibility |
| --- | --- |
| IPredication、IPredication&lt;T&gt; | Non-generic and strongly typed judgment contracts |
| Predication | Create implementation from delegate |
| PredicationBase&lt;T&gt; | Naming, parameter conversion and service matching |
| PredicationCollection | Combining multiple assertions |
| PredicationCombination | Short-circuit rules for AND or OR |

Returns success if the collection is empty; AND short-circuit on failure, OR short-circuit on success. Default object conversion affects weakly typed entries, and extensions should check parameter types, additional Parameters, and cancellation semantics.

## Actual Usage in the Log

The framework LoggerPredication makes judgments based on the log source, exception type and level. The configuration is read by Initialize. For specific implementation, see [LoggerPredication](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Diagnostics/LoggerPredication.cs). Such rules are suitable for output filtering and should not replace business authorization.

Use [Validators](validator.md) when you need to return the specific failure reason; use [Data conditions](../../data/conditions-and-operands.md) when you need to let the database filter records.
