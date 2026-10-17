# Change Impact Guide

| Change area / source | Dependencies and likely blast radius | Recommended regression checks |
| --- | --- | --- |
| User auth guard: `api/.../User/Controller/AbstractUserController.php` | Every User controller, login/profile/reset/game submit; session-backed identity. | Anonymous vs authenticated access; cross-account profile/reset; guest play and forged cookie. |
| User auth service: `api/.../User/Service/UserAuthenticationService.php` | Registration, verification, reset, profile edits, referrals, SMTP/social login, session data returned to UI. | Bcrypt, token expiry/one-use, validation, duplicate account, notification redaction, session invalidation. |
| Admin base controller: `api/.../Admin/Controller/AbstractAdminController.php` | All admin routes and staff operations. | Anonymous, admin, superadmin × CRUD operation matrix through direct API calls. |
| API routing: `module/{User,Admin}/config/module.config.php` | Browser/API route matching, controller factories, URL wrappers and deployment prefixes. | Route listing, supported HTTP verbs, route param constraints, frontend URL parity. |
| Common model/table abstraction: `Application/Model/AbstractModel.php`, `Application/Table/AbstractTable.php` | Serialization, input filters, write/delete logic and many API entities. | Entity validation/field-removal, soft-delete/audit, serialization of secrets, DB constraint behavior. |
| Game service dispatcher/scorers: `Application/Service/Games/*` | Nine games, user status, points, leaderboards, reward redemption. | Per-type correct/incorrect, tampered payload, duplicate submit, concurrent submissions, balances. |
| Commerce: `Application/Service/TransactionsService.php` | User redemptions, external orders, token cache, transaction history and mints. | Provider timeout/error/401, duplicate request, balance edge cases, reconciliation and idempotency. |
| Upload/image routing: API upload controllers, `S3Upload`, `ImagePath` | Profile/content/game images and UI URL construction. | MIME/extension/size, authorization, object key isolation, failed upload cleanup, URL rendering. |
| Web API client/config: `web/.../src/definitions/apis/client.js`, `src/config.js` | All public SSR/browser API calls, TLS, cookies, base URL. | TLS verification, CORS/session cookies, SSR production host, API error handling. |
| Admin API wrappers/config: `admin/.../src/api/`, `config.js` | Admin pages, payload/method, test/prod destination. | CRUD create/edit/delete, endpoint parity, session expiry, file upload. |
| Legacy game routing/controller: `game/.../application/config/routes.php`, `controllers/Welcome.php` | Old public content/game URLs, raw SQL, user session and DB writes. | Confirm active production route; SQL binding; scores/mints; route rewrites; no data divergence. |
| SQL migration history: `api/.../data/db/` | 70 API table targets and content/game/payment persistence. | Diff against sanitized baseline, migration order/rollback, FK/index/integrity checks on disposable DB. |
| Next build/deploy config: `web/.../next.config.js`, `amplify.yml`; admin equivalents | Redirects, build env and deployment artifact. | Route redirects, build/lint with pinned dependencies, correct API URLs and secret injection. |

Links and detailed relationships: [CONTROL_MAP.md](CONTROL_MAP.md), [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md), [SECURITY_AUDIT.md](SECURITY_AUDIT.md).