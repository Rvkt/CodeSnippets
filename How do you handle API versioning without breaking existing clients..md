
API versioning is handled by **keeping existing APIs backward-compatible** and introducing **new versions only for breaking changes**. Older and newer API versions run **in parallel**, allowing existing clients to continue working while new clients adopt updated contracts. Additive changes don’t require new versions, and old versions are **deprecated gradually** with monitoring and clear timelines to avoid forced upgrades or outages.

**One-line version:**

> “I avoid breaking existing clients by versioning only when necessary, running versions in parallel, and deprecating old APIs safely.”


## Scenario: Login API change without breaking old apps

#### Original API (already in production)

`POST /api/v1/login`

```json
{
  "token": "abc123",
  "expires_in": 3600
}

```

Thousands of users are on this version.  
You **must not break this**.

#### New requirement (breaking change)

You now need:
- refresh token
- user role
- device binding info

Changing `/api/v1/login` directly would break old apps.

##### Correct approach: introduce a new version
`POST /api/v2/login`
```json
{
  "access_token": "abc123",
  "refresh_token": "xyz456",
  "expires_in": 3600,
  "role": "ADMIN"
}
```

Now:
- Old apps keep calling `/v1/login`
- New apps call `/v2/login`

Both work in parallel.

## Backend handling
- Same authentication logic internally
- Different response serializers per version
- Separate validation rules if needed

No forced upgrades. No outages.