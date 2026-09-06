---
description: "Declarative identities, credentials, keys, verification codes and security base models of the Zongsoft.Security namespace in Zongsoft.Core."
icon: shield
---

# Zongsoft.Security

`Zongsoft.Security` is the security base layer in the core library. It is not directly equivalent to the complete security business module, but provides reusable abstractions, tools and runtime objects for the upper `Zongsoft.Security` plugin, Web authentication, business identity model and permission system.

The core library splits security capabilities into three categories:

| field | Main types | Purpose |
| --- | --- | --- |
| Claim identity | [`CredentialIdentity`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/CredentialIdentity.cs)、[`CredentialPrincipal`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/CredentialPrincipal.cs)、[`ClaimsIdentityModeling`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/ClaimsIdentityModeling.cs) | Represents identity, credential number, renewal token, scenario and identity model transition after login. |
| declaration tool | [`ClaimNames`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/ClaimNames.cs)、[`ClaimUtility`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/ClaimUtility.cs)、[`ClaimsIdentityExtension`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/ClaimsIdentityExtension.cs) | Adds namespace, description, authorization, type conversion, role and model mapping capabilities to standard claims. |
| security auxiliary | [`Password`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Password.cs)、[`Secretor`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Secretor.cs)、[`Certificate`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Certificate.cs)、[`ICaptcha`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/ICaptcha.cs) | Handles cryptographic digests, one-time secrets, certificates, signatures, human-machine identification, and challenge extensions. |

Permissions, authenticators, authorizers, users, roles, and member models are located at [`Zongsoft.Security.Privileges`](security/privileges.md). If you just want to know about permission trees, role inheritance, and authorization calculations, just read this page.

## Design Boundaries

The core security namespace only defines "how to express identity and security materials" and does not specify "how to design the user table" or "how to write the login interface". The default database, service implementation and plugin mounting are located in the [`Zongsoft.Security`](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Security) project; the business system can inherit the core abstraction, replace the model, query conditions, claims supplementary logic and permission storage fields.

The benefits of this layering are:

* The identity carrier is based on the standards [`ClaimsIdentity`](https://learn.microsoft.com/en-us/dotnet/api/system.security.claims.claimsidentity) _[( source code )](https://source.dot.net/#System.Security.Claims/ClaimsIdentity.cs)_ and [`ClaimsPrincipal`](https://learn.microsoft.com/en-us/dotnet/api/system.security.claims.claimsprincipal) _[( source code )](https://source.dot.net/#System.Security.Claims/ClaimsPrincipal.cs)_ and can integrate ASP .NET Core, token authentication and hosting context.
* Business identities can be restored from Claims to strongly typed models via _converters_ without the caller having to parse strings everywhere.
* Security materials such as credentials, verification codes, passwords, and certificates are exposed through interfaces, and the default implementation is replaceable. Business modules only rely on stable contracts.

## Declare Model

A claim is the smallest unit of identity information passed within a process after login. A [`Claim`](https://learn.microsoft.com/en-us/dotnet/api/system.security.claims.claim) _[Source](https://source.dot.net/#System.Security.Claims/Claim.cs)_ holds a fact such as user number, username, role, tenant number, language, or authorization information.

`ClaimNames` defines the declaration name of Zongsoft's additional convention:

| Name | meaning |
| --- | --- |
| `Namespace` | The namespace to which the identity belongs, often used for multi-tenant, organization or business domain isolation. |
| `Description` | Description text for the identity or claim object. |
| `Creation`、`Modification` | Creation and modification time. |
| `Authorization` | Authorization-related claims are usually treated as repeatable claims. |

`ClaimUtility` is responsible for converting between Claim values and .NET types. It identifies `bool`, [`DateTime`](https://learn.microsoft.com/en-us/dotnet/api/system.datetime) _[Source](https://source.dot.net/#System.Private.CoreLib/DateTime.cs)_, [`DateOnly`](https://learn.microsoft.com/en-us/dotnet/api/system.dateonly) _[Source](https://source.dot.net/#System.Private.CoreLib/DateOnly.cs)_, [`TimeOnly`](https://learn.microsoft.com/en-us/dotnet/api/system.timeonly) based on Claim's `ValueType` _[Source](https://source.dot.net/#System.Private.CoreLib/TimeOnly.cs)_, integer, float, [`TimeSpan`](https://learn.microsoft.com/en-us/dotnet/api/system.timespan) _[Source](https://source.dot.net/#System.Private.CoreLib/TimeSpan.cs)_ or Zongsoft type alias. When setting Claim, `ClaimsIdentityExtension.SetClaim(...)` will generate the appropriate `ValueType` in reverse.

Source: [src/Security/UserChallenger.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Security/UserChallenger.cs#L126) (excerpt; see source for context).

{% code title="UserChallenger.cs" %}
```csharp
private ClaimsIdentity Identity(UserProfile user)
{
	var identity = user.Identity(UserIdentity.Scheme, "Zongsoft");

	identity.SetClaim(nameof(UserProfile.SiteId), user.SiteId);
	identity.SetClaim(nameof(UserProfile.Gender), user.Gender);
	identity.SetClaim(nameof(UserProfile.Avatar), user.Avatar);
	identity.SetClaim(nameof(UserProfile.Grade), user.Grade);
	identity.SetClaim(nameof(UserProfile.TotalPosts), user.TotalPosts);
	identity.SetClaim(nameof(UserProfile.TotalThreads), user.TotalThreads);

	//进行其他声明定义
	this.OnClaims(identity, user);

	//返回新构建的身份
	return identity;
}
```
{% endcode %}

`ClaimsIdentityExtension` also provides common reading and judgment:

* `GetIdentifier<T>()` reads `ClaimTypes.NameIdentifier` and converts it to the specified type.
* `GetNamespace()` / `SetNamespace(...)` handle Zongsoft namespace declarations.
* `InRole(...)`, `InRoles(...)` and `IsAdministrator()` are judged by the role statement in the identity.
* `GetQualifiedName()` combines the namespace and name to form a `namespace:name` style qualified name.
* `AsModel<T>()` writes Claims to `IUser`, `IModel` or ordinary objects.

{% hint style="info" %}
Claims should hold "small facts that need to be read frequently after authentication," such as user number, name, tenant number, branch, language, role name, and licensing module. Don't stuff Claims with large objects, dynamic permission lists, or data that needs to be consistent in real time; these should typically be queried by the service or cached on demand.
{% endhint %}

## Credential Subject

`CredentialIdentity` is the login identity, and the name declaration and issuer declaration will be written during construction. The issuer is saved using `ClaimTypes.System`, so the same [`ClaimsPrincipal`](https://learn.microsoft.com/en-us/dotnet/api/system.security.claims.claimsprincipal) _[Source](https://source.dot.net/#System.Security.Claims/ClaimsPrincipal.cs)_ can look up the corresponding identity by authentication scheme or module.

`CredentialPrincipal` is the credential subject after login, extending the standard [`ClaimsPrincipal`](https://learn.microsoft.com/en-us/dotnet/api/system.security.claims.claimsprincipal) _[Source](https://source.dot.net/#System.Security.Claims/ClaimsPrincipal.cs)_:

| Properties | Description |
| --- | --- |
| `CredentialId` | Voucher number used for registration, lookup, logout, and model cache keys. |
| `RenewalToken` | Renewal token, used to refresh credentials or extend a session. |
| `Scenario` | Login scenarios, such as `web`, `api`, `mobile`. |
| `Validity` | How long the voucher is valid. |
| `Disposed` | Credential invalidation notification, the identity model cache will be invalidated accordingly. |

`CredentialPrincipal` can be serialized and deserialized, so the default credential provider can write it to cache, cookies, or other storage. After successful authentication, `Authentication.AuthenticateAsync(...)` will create the credential subject, execute _challenger (**C**hallengers) _, and finally register through `ICredentialProvider.RegisterAsync(...)`.

## Identity Model Transformation

`ClaimsIdentityModeling` is the currently recommended identity model entry. It obtains `CredentialPrincipal` from `ApplicationContext.Current.Principal` or the incoming _subject_, then takes out the corresponding identity according to the authentication scheme and performs conversion.

Converters come from two sources:

* `ClaimsPrincipalTransformer.Transformers` in `Authentication.Transformer`.
* `IClaimsIdentityTransformer` in the current application service container.

The conversion result will be cached with `CredentialId` and _scheme_ as keys; the cache will be invalidated when the credential body is released. UserIdentity.Current of Discussions reads the model by [Zongsoft.Discussions](https://github.com/Zongsoft/discussions/tree/main/src) scheme:

Source: [src/Security/UserIdentity.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Security/UserIdentity.cs#L110) (excerpt; see source for context).

{% code title="UserIdentity.cs" %}
```csharp
public static UserIdentity Current => ClaimsIdentityModeling.GetModel<UserIdentity>(Scheme);
```
{% endcode %}

The business _converter_ usually only handles the authentication schemes it knows, and converts _Claim_ into business identity attributes:

Source: [src/Security/UserIdentity.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Security/UserIdentity.cs#L130) (excerpt; see source for context).

{% code title="UserIdentity.cs" %}
```csharp
private bool OnTransform(UserIdentity user, Claim claim)
{
	switch(claim.Type)
	{
		case nameof(UserIdentity.SiteId):
			user.SiteId = (claim.Value != null && uint.TryParse(claim.Value, out var siteId)) ? siteId : 0;
			return true;
		case nameof(UserIdentity.Gender):
			user.Gender = (claim.Value != null && Enum.TryParse<Models.Gender>(claim.Value, out var gender)) ? gender : Models.Gender.None;
			return true;
		case nameof(UserIdentity.Avatar):
			user.Avatar = claim.Value;
			return true;
		case nameof(UserIdentity.Grade):
			user.Grade = (claim.Value != null && byte.TryParse(claim.Value, out var grade)) ? grade : (byte)0;
			return true;
		case nameof(UserIdentity.TotalPosts):
			user.TotalPosts = (claim.Value != null && uint.TryParse(claim.Value, out var totalPosts)) ? totalPosts : 0;
			return true;
		case nameof(UserIdentity.TotalThreads):
			user.TotalThreads = (claim.Value != null && uint.TryParse(claim.Value, out var totalThreads)) ? totalThreads : 0;
			return true;
		default:
			return false;
	}
}
```
{% endcode %}

## Passwords and Secrets

`Password` is the password digest structure recommended by the core library. It packages algorithms, exponents, random numbers and derived values in the same value, with a text format similar to:

Algorithm #index:random number | derived value. This is the format description, see [PasswordTest](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Security/PasswordTest.cs) for actual serialization and validation input.

`Password.Generate(...)` uses PBKDF2 to generate digests; defaults to SHA256 when the algorithm parameter is omitted. `Verify(...)` is recalculated based on the algorithm, random number, and exponent saved in the digest, so existing SHA1 digests can still be verified. The old `PasswordUtility` has been marked as obsolete, and new implementations should give priority to `Password`; `Passworder` in business services currently still retains the old format compatibility path, and will need to cooperate with heavy hash migration in the future.

`ISecretor` and `Secretor` are used for one-time secrets, such as SMS verification codes, email verification codes, password retrieval tokens, or secondary confirmations for sensitive operations. The default implementation saves secrets to distributed cache and supports:

| Ability | Description |
| --- | --- |
| `GenerateAsync(...)` | Generate secret by name and save additional text. |
| `VerifyAsync(...)` | The secret is verified but not deleted, which is suitable for scenarios that can be confirmed repeatedly within the validity period. |
| `RemoveAsync(...)` | Delete after successful verification, suitable for one-time consumption. |
| `Period` | Limit the minimum interval between repeated generation of the same name. |
| `Transmitter` | To send secrets, you can integrate text messages, emails, site messages and verification code verification. |

{% hint style="warning" %}
The verification code name should include the business scenario and target identification. You can refer to Secretor.Transmitter.GetKey to combine the scheme, destination, template, scenario and channel, and ensure global uniqueness. Excessively wide names may cause different users or different scenarios to cover each other.
{% endhint %}

## Certificates, Signatures and Challenges

`ICertificate`, `ICertificateProvider<TCertificate>`, `ICertificateResolver`, and `Certificate` encapsulate certificate identification, issuer, subject, validity period, and RSA/X509 certificate adaptation. `ISignaturer` and `ISecretor` cover signature and secret verification scenarios respectively.

`IChallenger` is the supplementary questioning point after certification. The authenticator is only responsible for "whether the credentials are correct" and "issuance of basic identities", while the challenger is responsible for business inspection and claims enhancement after login. Discussions' UserChallenger does the following:

* Read the user number from the primary identity and query or create a Discussions user profile.
* SiteId is determined from the identity namespace when creating a profile; existing profiles retain their own site information.
* Call OnVerify extension point, and then create the [Zongsoft.Discussions](https://github.com/Zongsoft/discussions/tree/main/src) identity to join the subject.
* Write SiteId, Gender, Avatar, Grade, TotalPosts, and TotalThreads statements.

Currently OnVerify is an empty implementation and cannot be used to claim verified account activation status, site status, or permission scope; these business admission rules need to be supplemented by actual modules. The data statistics statement is also a snapshot at the time of issuance and will not be automatically refreshed with each post.

This split allows different authentication methods such as password login and verification code login to share the same set of business challenge rules.

## Exceptions and Causes

All security related exceptions are inherited from `SecurityException`. `SecurityReasons` provides a stable reason code, for example:

| Reason | Common scenarios |
| --- | --- |
| `InvalidIdentity` | The identity does not exist, the number is invalid, or the identity cannot be issued. |
| `InvalidPassword` | Wrong password. |
| `AccountDisabled`、`AccountSuspended` | The account is unavailable or too many attempts have been made. |
| `Forbidden` | After passing the certification, the business access conditions are not met. |
| `VerifyFaild` | Verification code or secret verification failed. |

The business code should give priority to throwing `AuthenticationException` or `AuthorizationException` with reason code to facilitate unified processing by the web layer, logs, monitoring and front-end.

## Related Pages

* [Zongsoft.Security.Privileges](security/privileges.md)
* [security module overview](../security.md)
* [Plugin Framework](../plugins/README.md)
* [Data Access Interfaces](../data/data-access.md)
