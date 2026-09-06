---
description: "Understanding the results, failure information, and calling context of the password policy validator from the framework security module."
icon: clipboard-check
---

# Validators

IValidator<T> returns whether it passed and allows the reason to be reported via the failure callback. Discussions' DataValidator implements the data access validation contract and is responsible for site and audit fields; the two cannot be confused just because of similar names.

## Real Password Validator

Source: [framework/Zongsoft.Security/src/Validators/PasswordValidator.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Validators/PasswordValidator.cs#L42) (excerpt; see source for context).

{% code title="PasswordValidator.cs" %}
```csharp
[Service(typeof(IValidator<string>))]
public class PasswordValidator : IValidator<string>, IMatchable
```
{% endcode %}

The security module registers PasswordValidator for service discovery. The policy is not hard-coded as "at least eight bits" but is provided by the IdentityOptions parameter.

Source: [framework/Zongsoft.Security/src/Validators/PasswordValidator.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Validators/PasswordValidator.cs#L58) (excerpt; see source for context).

{% code title="PasswordValidator.cs" %}
```csharp
var options = parameter as Configuration.IdentityOptions;

//如果没有设置密码验证策略，则返回验证成功
if(options == null || options.PasswordLength < 1)
	return true;

//如果如果密码长度小于配置要求的长度，则返回验证失败
if(string.IsNullOrEmpty(data) || data.Length < options.PasswordLength)
{
	failure?.Invoke($"The password length must be no less than {options.PasswordLength} characters.");
	return false;
}

bool isValidate;
```
{% endcode %}

If no strategy is provided or the length is less than one, success is returned; if the length is insufficient, the reason is returned via failure. This is an important boundary of the current implementation. Deployers must provide policies that meet their own requirements. Unconfigured policies cannot be understood as adopting a default strong policy.

## Strength and Failure Messages

The subsequent logic determines pure numbers, lowest, normal and highest intensity respectively according to the configuration. See the same source file for the complete method. The caller can collect failure messages and display them to the user, or it can only care about the return value; do not write the entered password itself into the log.

## Synchronization, Asynchronousness and Context

The verification interface has synchronous and asynchronous forms, and custom parameters can be passed. Currently password checks are computed locally; validators that need to call external dependencies should propagate the cancellation token and control the cost of the call. Successful synchronization does not mean that persistence or authentication has been successful, these are subsequent business steps.

| demand | The mechanism that should be used |
| --- | --- |
| Check whether the input value complies with the rules and return the reason | IValidator |
| Whether the conditions are met | [Predication](predication.md) |
| Data query site constraints and audit fields | [Discussions DataValidator](../../data/services.md) |
| Can the user perform resource actions? | [Authentication and Authorization](../../security/authentication.md) |

The calling location can be checked against frame [UserServiceBase.Password](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/UserServiceBase.Password.cs).
