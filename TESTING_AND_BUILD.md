# Testing and Build

## Available Commands

| Module | Install / run | Build / lint / tests | Notes |
| --- | --- | --- | --- |
| Admin | From `admin/ravi_bollymints-admin-9df7f773dd7c/`: `npm install`; `npm run dev` (script binds port 8080). | `npm run build`, `npm run lint`; no test script. | Needs API URL and backend/session CORS settings. |
| Web | From `web/ravi_bollymints-web-97661d85ac77/`: `npm install`; `npm run dev`. | `npm run build`, `npm run lint`; no test script. | Next.js 10.2 / React 17; API URL defaults to test host. |
| API | From `api/ravi_bollymints-api-25c52a20b786/`: `composer install`; `composer serve`. | `composer test`, `composer cs-check`, `composer static-analysis`. | Requires configured SQL connection and PHP extensions/packages. These scripts are in Composer manifest. |
| Legacy Game | PHP/Apache deployment rooted at the project; `index.php` and `.htaccess` are present. | No application build/test script found. | Local server, rewrite config, PHP compatibility, database, and writable paths need environment-specific setup. |

The extracted tree currently has no API `vendor/` or frontend `node_modules/`, so these commands were not run. No dependencies were installed as part of this documentation-only audit.

## Existing Coverage

- API: one visible PHPUnit controller test for the Laminas skeleton index route, not domain, auth, CRUD, payment, upload, or game behavior.
- Admin/Web: package scripts define lint/build only; no project-owned unit/integration/E2E tests were found.
- Legacy CodeIgniter: no application-owned tests were found in the inspected tree. Vendored libraries include their own unrelated test configuration.

## Verification Priorities

1. Add feature tests for unauthenticated profile/reset attempts, admin authorization, and session logout.
2. Test game scoring against forged client flags, repeated submissions, concurrency, and points consistency.
3. Test transaction idempotency/provider failure paths and upload MIME/size/access controls.
4. Add UI/API contract and page-flow coverage for registration/reset, admin CRUD, game play, rewards, and purchase.
5. Validate migrations against an isolated local schema fixture; never use production credentials for tests.

Build/deployment artifacts are evidence of possible workflows only. There was no production config, test database, or CI job execution in this audit.