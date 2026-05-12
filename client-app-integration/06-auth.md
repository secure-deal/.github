# Authentication

## Login contract

Route:

`POST /api/client/auth/login`

Body:

```json
{
  "bank_id": "bank-user-12345",
  "phone": "+992901234567",
  "name": "Rahimov Farrukh"
}
```

## Session model

- Backend sets auth cookies on login.
- Frontend should not manage bearer tokens manually for the buyer app.
- All authenticated API requests must include cookies.

## Session endpoints

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/api/client/auth/login` | Start or refresh buyer session. |
| `GET` | `/api/client/auth/me` | Current buyer profile/session bootstrap. |
| `POST` | `/api/client/auth/logout` | Revoke current session and clear cookies. |
| `PUT` | `/api/client/auth/avatar` | Update avatar URL. |
| `DELETE` | `/api/client/auth/avatar` | Remove avatar URL. |

## Frontend rules

1. On app start, call `/me` if a session may exist.
2. On `401`, redirect to login or rerun the bootstrap path.
3. On logout, clear local caches that contain buyer-specific data.
4. Do not treat login response body as the session payload; the authoritative session comes from `/me`.
