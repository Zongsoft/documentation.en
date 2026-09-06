---
description: "Permission model, authenticator, authorizer and default implementation of Zongsoft.Security.Privileges in Zongsoft.Core."
icon: user-shield
---

# Zongsoft.Security.Privileges

`Zongsoft.Security.Privileges` is the identity authentication and permission model namespace in the core library. It defines users, roles, memberships, permission definitions, authenticators, authorizers, permission services and permission calculation rules; the default database implementation is located at [Zongsoft.Security Project](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Security), and business systems can inherit these base classes and replace the storage model.

## Core Concept

The design of Privileges is not to hard-code permissions on interfaces, controllers or pages, but to separate "permission definitions" and "authorization results":

| concept | Type | Description |
| --- | --- | --- |
| Permission definition | [`Privilege`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/Privilege.cs)、`Privilege.Permission` | The permission tree declared by the plugin explains the authorizable capabilities of the system. |
| Authorization record | [`IPrivilege`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/IPrivilege.cs)、[`IPrivilegable`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/IPrivilegable.cs) | The database stores "what permissions a user or role has been granted/denied". |
| Authorization target | `Privilege.Permission.Target` | Business resources or function points are declared by specific modules and cannot be inferred solely from the data table name. |
| Authorization operation | `Privilege.Permission.Action` | For target actions, such as `Get`, `Query`, `Create`, `Update`, `Delete`, no-operations are handled as `*`. |
| Authorization subject | [`IUser`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/IUser.cs)、[`IRole`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/IRole.cs)、[`Member`](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/Member.cs) | Users, roles, and membership relationships after the user or role joins the role. |

In one sentence: the plugin is responsible for declaring "what permissions the system has", the database is responsible for recording "who is granted or denied what", and the runtime is responsible for calculating the final result based on role inheritance and denial rules.

## Permission Tree

The permission tree consists of `PrivilegeCategory` and `Privilege`. Classification is used for navigation and organization, and permissions are used to express an authorizable capability; a permission can contain multiple `Permission`, that is, multiple combinations of goals and actions.

Discussions There is currently no independent permission definition tree declared. It directly determines the moderation status, author and moderator relationships during topic reading; it cannot be assumed that adding a permission name will automatically change these business rules. The assembly mechanism of the permission tree itself can read the core [PrivilegeCategory](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/PrivilegeCategory.cs) and [recognizer](../components/discriminator.md).

Permission definition and permission checking are two steps: the definition tells the configuration interface what authorizable capabilities are available, and the check is responsible for calculating whether the subject is granted permission before actually executing the operation. PermissionCollection supports target and action matching, and empty actions are processed as wildcard actions; the authorization status of named permissions and data records still needs to be associated through services.

## Permissions Localization

`Privilege` and `PrivilegeCategory` will read the title and description from the resource as agreed. Resource keys are combined according to conventions such as permission name, classification path, Category and Description. The specific search order is based on [Privilege](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/Privilege.cs) and [PrivilegeCategory](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/PrivilegeCategory.cs). The name is used for stable matching, and the resource text is responsible for display; Discussions See [Resource management](../resources.md) for the organization of existing business resources.

## Users, Roles and Members

Both `IUser` and `IRole` inherit Zongsoft's identifiable model. The core attributes include name, enabled status, avatar, nickname, namespace and description. `IUser.Administrator` and `IRole.Administrators` are the administrator names agreed by the framework.

`Member` represents role members. Members can be users or roles:

| member type | meaning |
| --- | --- |
| `MemberType.User` | User joins role. |
| `MemberType.Role` | Roles join roles to form role inheritance. |

The `Member` table in the default database uses `RoleId + MemberId + MemberType` to store membership relationships. The UserIdentity of Discussions implements IUser and adds discussion business information such as SiteId. Its forum membership relationships are still maintained by the module's own model.

## Authentication Link

The authentication entry is `Authentication.AuthenticateAsync(...)`:

1. Trigger the Authenticating event and find `IAuthenticator` from `Authentication.Authenticators` according to the scheme.
2. Call `VerifyAsync(...)` to verify the input credentials and return the ticket.
3. Call `IssueAsync(...)` to issue the ticket as [`ClaimsIdentity`](https://learn.microsoft.com/en-us/dotnet/api/system.security.claims.claimsidentity) _[Source](https://source.dot.net/#System.Security.Claims/ClaimsIdentity.cs)_.
4. Create `CredentialPrincipal`, carrying `CredentialId`, `RenewalToken`, `Scenario` and `Validity`.
5. Execute `Authentication.Challengers` in sequence.
6. Register the credential via `Authentication.Authority`.
7. Triggers the Authenticated event; exception branches also trigger this event and carry errors.

The core library provides two authenticator base classes:

| base class | Default scheme | input | Applicable scenarios |
| --- | --- | --- | --- |
| `Authentication.IdentityAuthenticatorBase` | empty string | namespace, identity, password | Log in with username, email, mobile phone number and password. |
| `Authentication.SecretorAuthenticatorBase` | `Secret` | Verification code or one-time secret | Log in with SMS verification code, log in with email verification code, and confirm password retrieval. |

The default security plugin mounts the implementation to `/Workbench/Security/Authentication`:

Source: [framework/Zongsoft.Security/src/Zongsoft.Security.plugin](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Zongsoft.Security.plugin#L55) (excerpt; see source for context).

{% code title="Zongsoft.Security.plugin" %}
```xml
<!-- 挂载身份验证器 -->
<extension path="/Workbench/Security/Authentication">
	<object name="Identity" value="{static:Zongsoft.Security.Privileges.Authenticators.Identity, Zongsoft.Security}" />
	<object name="Secretor" value="{static:Zongsoft.Security.Privileges.Authenticators.Secretor, Zongsoft.Security}" />
</extension>
```
{% endcode %}

## Business Inquiry

`IChallenger` is executed after the authenticator issues the identity, and is suitable for business access rules and claims enhancement. The business system places site differences in `UserChallenger` of each site and mounts them to `Authentication.Challengers` through plugins:

Source: [src/Zongsoft.Discussions.plugin](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Zongsoft.Discussions.plugin#L40) (excerpt; see source for context).

{% code title="Zongsoft.Discussions.plugin" %}
```xml
	<!-- 挂载身份质询器 -->
	<extension path="/Workbench/Security/Authentication/Challengers">
		<object value="{static:Zongsoft.Discussions.Security.UserChallenger.Instance, Zongsoft.Discussions}" />
	</extension>

	<!-- 挂载身份转换器 -->
	<extension path="/Workbench/Security/Authentication/Transformers">
		<object value="{static:Zongsoft.Discussions.Security.UserIdentity+Transformer.Instance, Zongsoft.Discussions}" />
	</extension>
```
{% endcode %}

Discussions' UserChallenger appends the site identity and writes claims such as SiteId, avatar, rating, and post statistics. UserIdentity.Transformer only recognizes the [Zongsoft.Discussions](https://github.com/Zongsoft/discussions/tree/main/src) authentication scheme and converts these claims into business identities; business code reads them through UserIdentity.Current. For the complete source code and field meanings, see [security basics](../security.md).

## Authorization Record

Authorization records are read and written by `IPrivilegeService`. The default database has two authorization tables:

| table | function |
| --- | --- |
| `Privilege` | Save the authorization method of a user or role for a certain permission name. |
| `PrivilegeFiltering` | Save fields or data filtering expressions under a certain permission. |

The authorization method is represented by `PrivilegeMode`:

| value | meaning |
| --- | --- |
| `Granted` | Expressly granted. |
| `Denied` | If explicitly rejected, the calculation will be higher than the grant at the same level. |
| `Revoked` | Withdrawn or empty status, default implementations usually treat it as if there is no valid authorization. |

The default security plugin provides generic permission storage. Discussions does not add permission table overrides in the form of TenantId and BranchId; its site query boundary is implemented through DataValidator, and the two mechanisms cannot be confused.

Discussions The independent role permission writing example is not implemented; the default database writing path is shown in [PrivilegeService](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Security/src/Privileges/PrivilegeService.cs) and [PrivilegeServiceBase](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/PrivilegeServiceBase.cs). The actual user or role identity is used when calling, and the update scope of the record is explicitly authorized by the business portfolio layer.

## Permission Calculation

The authorization judgment entry is `IAuthorizer.AuthorizeAsync(identity, privilege, parameters, cancellation)`. By default, `AuthorizerBase` will first cache the calculation results by user ID, and then check whether the specified permission name is in the final set.

The default calculation process of `PrivilegeEvaluator` is:

1. Convert the current user or role ID to a member ID.
2. Use `IMemberService.GetAncestorsAsync(...)` to take out all ancestor characters and return them by level from far to near.
3. Read the authorization records of each group of roles hierarchically.
4. Read the current user or role's own authorization record.
5. Call `PrivilegeEvaluatorBase` to collapse all declarations.

The folding rule is "nearest priority, peer rejection priority":

* The ancestor level enters the context first, and the current user or current role enters the context last, so authorizations closer to the current subject are processed later.
* In the same level, `Denied` will remove the permission with the same name and prevent `Granted` at the same level from rejoining.
* If the next level `Granted` again, it can override the rejection of the further level; this is "nearest priority".

Source: [framework/Zongsoft.Core/src/Security/Privileges/AuthorizerBase.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Privileges/AuthorizerBase.cs#L67) (excerpt; see source for context).

{% code title="AuthorizerBase.cs" %}
```csharp
public virtual async ValueTask<bool> AuthorizeAsync(ClaimsIdentity user, string privilege, Parameters parameters, CancellationToken cancellation = default)
{
	if(user == null)
		return false;

	if(privilege == null)
		return false;

	var privileges = await _cache.GetOrCreateAsync(user.Identify(),
		key => (GetPrivilegesAsync((Identifier)key, cancellation), TimeSpan.FromMinutes(60)));

	return privileges.Contains(privilege);

	async Task<HashSet<string>> GetPrivilegesAsync(Identifier identifier, CancellationToken cancellation)
	{
		var privileges = new HashSet<string>(StringComparer.OrdinalIgnoreCase);

		var results = this.Evaluator.EvaluateAsync(identifier, parameters, cancellation);
		await foreach(var result in results)
			privileges.Add(result.Privilege);

		return privileges;
	}
}
```
{% endcode %}

{% hint style="warning" %}
The default `AuthorizerBase` caches the final permission set with the user ID, using a 60-minute sliding expiration. After modifying role members or authorization records, business implementation needs to consider cache refresh and the cache invalidation mechanism owned by the application, otherwise the old authorization results may be seen in a short period of time.
{% endhint %}

## Permission Filtering

`IPrivilegeService.Filtering` provides permission filtering services. It does not determine "whether there is a certain permission", but describes "which fields should be hidden or what scope should be restricted when the permission is given." The default database's `PrivilegeFiltering.PrivilegeFilter` is a string expression. Discussions does not integrate this set of authorization filter expressions; its PostFilter and ThreadFilter handle text visibility and belong to the data filter mechanism. See [Filters](../components/filter.md) for the real implementation.

This capability is suitable for data query, export or field tailoring of the details interface. The caller should first complete the normal authorization, then read the filtering service and hand the filtering expression to the data layer or business layer for interpretation.

When extending the default implementation, the model, authorization record storage, subject identity, and cache invalidation should be checked separately; simply replacing the user model will not automatically change all permission query conditions.

## Usage Suggestions

* When adding function permissions, first declare the classification, permissions and target actions in `*-privileges.plugin` of the corresponding business plugin, and then fill in the resource text.
* The authorization record only saves the permission name and authorization method. Do not copy the permission tree structure.
* Multi-tenant systems should bring tenant boundaries into the data conditions of permission services to avoid reading authorization records across tenants.
* Tenants, organizations, role names and license summaries are saved in Claims; real-time sensitive authorization results are still queried through `IAuthorizer` or `IPrivilegeService`.
* Don't go too deep into role inheritance. The deeper the hierarchy, the harder it is to interpret authorization and the easier it is to miss cache invalidations.
* `Denied` is used to explicitly block inherited permissions; for normal removal, use `Revoked` or delete the authorization record.

## Related Pages

* [Zongsoft.Security](../security.md)
* [Plugin Manifests and Loading](../../plugins/plugin-file.md)
* [Data Access Interfaces](../../data/data-access.md)
* [Categories](../collections/category.md)
