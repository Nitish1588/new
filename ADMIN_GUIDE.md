# Admin Guide

## Architecture and Entry

The admin UI is its own Next.js application. Page routes live under `admin/.../pages/`, API wrappers under `src/api/`, reusable forms under `src/components/`, and its default API base URL is in `config.js` with an optional local override. The backend is the Laminas `/admin` route tree and `Admin` module.

Login posts email/password to `/admin/authentication/login`; the controller stores the returned admin model in a Laminas session container. Subsequent requests use Axios `withCredentials`. `logged-in-details` reads the session value; logout removes it. The route base exempts login and type lookups in `AbstractAdminController`.

## Feature Areas

| Admin UI | Backend route family | Main data area |
| --- | --- | --- |
| Dashboard | `/admin/dashboard` | Aggregates across content/game data. |
| Articles/subarticles | `/admin/articles`, nested `/subarticles` | `articles`, `sub_articles`, celebrity join. |
| Blogs, facts, banners, records | `/admin/blogs`, `/do-you-know`, `/banners`, `/records-archive` | Corresponding content tables. |
| Movies/celebrities | `/admin/movies`, `/admin/celebs` | Film/celebrity tables and join/lookups. |
| Games | `/admin/games/{game_type}` | Game definitions, type-specific details/questions, audit data. |
| User/admin lists | `/admin/users`, `/admin/admins` | `users`, `admins`, identity lookups. |
| Transactions/subscriptions | `/admin/transactions`, `/admin/subscriptions` | Commerce/account activity tables. |
| Metadata and weekly poll | `/admin/metatags`, `/admin/poll-week-questions` | Metadata and poll tables. |
| Upload | `/admin/uploads/image` | S3-backed media path; exact upload security policy needs runtime review. |

## Authorization and Settings

- Admin base controller checks that an `Admin` session container is set, except for selected path substrings.
- Login accepts only verified admin/superadmin role ids in the auth service.
- `AbstractAdminController` declares CRUD permission bits, but code search found no `checkPermission()` invocation outside its own definition. Treat role-based restriction beyond login as absent until each endpoint is tested.
- Content records are generally database-backed. No evidence supports calling an admin setting an environment or deployment setting.
- Uploaded images use S3 helper/config. Secret values are intentionally not documented here.

## Safe Extension Path

For a new admin resource, trace an existing resource such as Articles: add route/controller factory in the Admin module, implement controller-to-service calls, reuse/create a model and TableGateway, add a wrapper in `src/api/`, then add the page/form under `pages/` and `src/components/`. Add authorization checks at the backend before shipping; the current permission helper is not automatically applied. Regression checks should cover anonymous access, both admin roles, validation, soft-delete/audit behavior, and the public consumer of modified content.