# M1 — Auth & Workspace UI

**Status:** 🔜 Next  
**Depends on:** M0 (shipped)  
**Reference:** [docs/ROADMAP.md — Milestone 1](../ROADMAP.md)

---

## Problem

Currently the platform requires manual SQL to bootstrap:
- A workspace row
- A project row
- An integration row
- An ingestion key (HMAC-hashed, inserted with `key_prefix`)

There is no login, no session, and no UI for key management. This blocks any real user from self-hosting without direct database access.

---

## Scope

### 1. Workspace signup

- `/signup` page: workspace name + admin email + password
- Creates `workspaces`, `users`, `workspace_members` rows in one transaction
- Returns a session token (JWT or cookie)

### 2. Login / session

- `/login` page: email + password
- Session persists across admin reloads (JWT stored in `localStorage` or `httpOnly` cookie)
- Logout clears session

### 3. Key management UI

- Settings → API Keys page
- List existing ingestion keys (prefix shown, full key never stored)
- Generate new key: shows full key once on creation
- Revoke key (sets `revoked = true`)

### 4. Admin auth middleware

- Admin API calls must include the user's session token, not just the ingestion key
- Add a `/api/v1/auth/me` endpoint returning workspace context
- Protect all admin-facing analytics/project routes with session auth
- Keep ingestion routes protected by ingestion key auth (unchanged)

### 5. Project management UI

- Projects page: create / rename / archive projects
- Each project shows its `id` (needed for SDK `projectId` config)

---

## Files to create / modify

| Path | Change |
|---|---|
| `apps/admin/src/pages/Login.tsx` | New login page |
| `apps/admin/src/pages/Signup.tsx` | New signup page |
| `apps/admin/src/pages/Settings.tsx` | Add API Keys section |
| `apps/admin/src/context/AuthContext.tsx` | Session state, `useAuth()` hook |
| `apps/api/src/routes/auth.ts` | `POST /api/v1/auth/signup`, `/login`, `/logout`, `/me` |
| `apps/api/src/middleware/session.ts` | JWT verify middleware for admin routes |
| `apps/api/src/routes/keys.ts` | `GET/POST/DELETE /api/v1/keys` |
| `packages/database/src/migrations/006_auth.ts` | Add `password_hash` to `users`, `sessions` table |

---

## Exit criteria

- New user can sign up, log in, and see the dashboard with no SQL required
- User can create and revoke ingestion keys from the UI
- The generated key can be used in the SDK `init()` call immediately
- Workspace A cannot access workspace B's data
- Admin API calls fail with 401 if session is expired
