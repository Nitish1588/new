# Developer Onboarding

## Orientation

This workspace contains four independent application roots nested inside the ZIP extraction folders. Start with `web/.../pages/` for public UI work, `admin/.../pages/` for staff UI, `api/.../module/` for backend behavior, and `game/.../application/` only when a task explicitly involves the legacy CodeIgniter path. Confirm whether that legacy path is live before making a behavior change.

Typical public flow: Next page → `src/definitions/apis/` wrapper → Laminas `/user` controller → shared service → TableGateway/model → SQL. Admin follows the same pattern through `src/api/` → `/admin`. Exact shared-database and deployment relationships are not verified.

## Prerequisites and Local Setup

- Node.js/npm compatible with the locked Next versions; PHP/Composer compatible with the API's PHP constraint; database service and extensions required by the configured driver.
- Do not use production credentials. The API uses `config/autoload/local.php` copied/configured from `.dist`; the legacy app uses CodeIgniter `application/config/database.php` and related local settings.
- The code has no confirmed `.env` contract. API config keys visible in templates include database `db`, `website_url`, `qc_url`, commerce credential keys, encryption settings, upload/S3 keys; populate only from a secure local source. Values are intentionally omitted.
- Frontends select API base URLs from JavaScript config files; the checked-in default is a test API endpoint. Admin's local override is `config-local.js`; web has a local config import in `src/config.js`.

Commands from each module root, supported by its manifest/source:

```sh
# admin or web
npm install
npm run dev
npm run build
npm run lint

# API
composer install
composer serve
composer test
composer cs-check
composer static-analysis
```

No dependency installation or app startup was performed in this audit. API commands require a valid local DB/config; frontend flows require API connectivity and credentialed CORS. The legacy app additionally requires Apache rewrite support or equivalent routing and writable runtime paths.

## Common Changes

- **Add a public page:** add a file under `web/.../pages/`, reuse components in `src/components/`, and use an existing wrapper in `src/definitions/apis/`; inspect `_app.jsx`, route guards, loading/error handling, and page metadata.
- **Add an API endpoint:** register it in the correct User/Admin `module.config.php`, add a controller factory/action, delegate business logic to an `Application/Service`, use a model/TableGateway, and add request/response tests. Define auth/policy explicitly; do not rely on the current base guard without checking it.
- **Change a response:** update service transformation/model serialization and all public/admin consumers. Check `modelForGet()` field filtering and response envelope conventions.
- **Change schema:** identify all 70 gateway targets and both legacy/new migration folders, obtain canonical DDL, add a forward-only migration with rollback strategy, update model/filter/Table code, and test on a disposable database. Never run archived migration scripts against production.
- **Change admin permissions:** audit `AbstractAdminController` and every action; current `checkPermission()` is not automatically invoked. Add direct API policy tests.
- **Change game rules:** update the API-specific scorer and tests, then check whether the legacy CodeIgniter implementation is active and has a duplicate rule. Verify points and game status consistency.
- **Change configuration/external integration:** locate the relevant local config key and service use; keep values in secret storage. Verify timeout, retries, idempotency, TLS, and redacted logging.
- **Fix a bug:** reproduce with a focused API/UI test, trace page → API wrapper → route → controller/service → table, then add a regression test at the responsible layer.

## Regression Checklist

- Anonymous, regular-user, admin, and superadmin API behavior is intentional and tested.
- The API/UI request path, fields, method, response envelope, and cookie/CORS contract agree.
- Database writes preserve FK/audit/soft-delete expectations and transaction consistency.
- Game results are recomputed server-side; duplicate submission and point totals are checked.
- External calls handle failure/retry without duplicate purchase or data leakage.
- Build/lint/tests pass for all touched modules; legacy behavior is verified if in scope.