# Project Overview

## Purpose

**CONFIRMED by source naming and implemented pages/services:** Bollymints provides Bollywood movies, celebrities, articles/blogs, user accounts, browser games, game points (“mints”), memberships/rewards, and an administrative content workflow. The public pages are in `web/.../pages/`; API domain services and controllers are under `api/.../module/`; the legacy CodeIgniter application has its own public entry point and routes under `game/.../`.

## Technology

| Area | Evidence | Confidence |
| --- | --- | --- |
| Public UI | Next.js 10.2.0, React 17, Redux Toolkit, Axios, MUI, Formik/Yup, Firebase/social-login packages in the public package manifest. | CONFIRMED by manifest; actual runtime use varies by package. |
| Admin UI | Next.js, React, Axios, MUI, Formik/Yup, rich-text editors. Its manifest uses `latest` for key packages, so resolved versions depend on the lockfile. | CONFIRMED by manifest. |
| API | PHP 7.3/8.0 constraint, Laminas MVC 3, Laminas DB/TableGateway, sessions, input filters; SQL DDL uses MySQL/InnoDB syntax. | CONFIRMED by Composer and source. |
| Legacy app | PHP CodeIgniter-style `index.php`, `application/config/routes.php`, controllers/models/views, MySQLi database config. Exact framework version/runtime compatibility is unverified. | CONFIRMED framework family; version UNVERIFIED. |
| Storage/integrations | API source integrates S3 uploads, SMTP email, Google/Facebook login, and an external catalog/order service. | CONFIRMED in code; deployed credentials/connectivity UNVERIFIED. |
| Deployment | Amplify YAML exists in both Next apps; API has Docker/Vagrant artifacts; legacy site has Apache rewrite configuration. | Configuration exists; production topology UNVERIFIED. |

## Modules and Boundaries

- `admin/` and `web/` are independent Next.js projects with separate package manifests and separate Axios clients.
- `api/` has Laminas `Admin`, `User`, and `Application` modules and MySQL-style table gateways.
- `game/` is a separate legacy application with its own config, route table, controllers, models, and bundled CodeIgniter/system libraries.
- The web/admin default API URLs point to `/api/public/user` and `/api/public/admin` on a test API host. This suggests an API mount/rewrite prefix not defined by the Laminas route configs themselves.
- **UNVERIFIED:** whether legacy `game/` shares the API database, whether it is still deployed, and whether it is the canonical game runtime.

## Top Risks

See [SECURITY_AUDIT.md](SECURITY_AUDIT.md): unauthenticated account mutation/reset paths, source-embedded credentials, disabled TLS verification in the public web API client, raw SQL interpolation in legacy gameplay, and an unused admin permission helper.