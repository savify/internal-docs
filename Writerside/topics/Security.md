# Security

## Authentication

Authentication is implemented using JWT Token and Bearer scheme using IdentityServer. For now, only one authentication
method is implemented: sending email/password in JSON payload.

Authenticated user get access and refresh tokens. Access tokens are short-term, so Front-End should constantly refresh it
with refresh token.

## Authorisation

Authorization is achieved by implementing **RBAC (Role Based Access Control)** using Permissions. Permissions are more granular
and a much better way to secure your application than Roles alone. Each User has a set of Roles and each Role contains one
or more Permission. The User's set of Permissions is extracted from all Roles the User belongs to. Permissions are always
checked on Controller level - never Roles:

```C#
[HttpPost]
[HasPermission(BanksPermissions.ManageBanks)]
[ProducesResponseType(StatusCodes.Status201Created)]
public async Task<IActionResult> SynchroniseBanks()
{
    var result = await _banksModule.ExecuteCommandAsync(new SynchroniseBanksCommand(_executionContextAccessor.UserId));

    return Created("", new
    {
        result.Status
    });
}
```

If endpoint should not be secured, you should use `[AllowAnonymous]` and `[NoPermissionRequired]` attributes:
```C#
[AllowAnonymous]
[NoPermissionRequired]
[HttpPost]
[ProducesResponseType(StatusCodes.Status201Created)]
public async Task<IActionResult> RequestPasswordReset(RequestPasswordResetRequest request)
{
    var passwordResetRequestId = await _userAccessModule.ExecuteCommandAsync(
        new RequestPasswordResetCommand(request.Email));

    return Created("", new
    {
        Id = passwordResetRequestId
    });
}
```

