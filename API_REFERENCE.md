# API Reference

## Scope and Route Counts

The Laminas route configs contain **67 active route patterns** across the `Application`, `Admin`, and `User` module configs; route patterns include optional segments and nested child routes. The admin and user controller trees contain **139 active public handler declarations** matching REST methods or `*Action` methods. These are source counts, not a normalized number of unique `(HTTP method, full URL)` endpoints: Laminas REST dispatch and optional `/:id`/`/:action` patterns expand into multiple operations, and not every action is constrained to one HTTP method in route config.

The route roots are `/admin` and `/user`. Frontend defaults mount them under `/api/public/admin` and `/api/public/user` respectively; the proxy/base prefix is outside the Laminas route definitions. Confirm the deployed reverse-proxy configuration before using a public base URL.

## Common Contract

- Admin client base URL: `BASE_URL` from `admin/.../config.js`, optionally overridden by `config-local.js`; checked-in default is a test API host.
- Web client base URL: `BASE_URL` from `web/.../src/config.js`; checked-in default is a test API host.
- Both clients set `withCredentials=true`; sessions are cookie-backed. The web client additionally disables Node TLS certificate verification; see [SECURITY_AUDIT.md](SECURITY_AUDIT.md).
- Success JSON: `success: true`, `data`, `success_msg` (array), `success_code`. Error JSON: `success: false`, `error_msg` (array), `error_code`. These body codes do not establish actual HTTP status codes.
- POST bodies are JSON-decoded by `AbstractService`; PUT bodies use URL-encoded `parse_str`. File uploads are handled by upload controller/service and need multipart content; the exact runtime limit/MIME behavior was not verified.
- Most CRUD methods delegate validation to model `getInputFilter()` and persistence to `AbstractTable`. Required fields differ per model; this reference does not claim a universal payload schema.

## Admin Route Inventory

All paths below are relative to `/api/public`; path templates are exactly the active route shapes (optional parts shown in brackets).

| Route | Handler methods/actions discovered | Authentication / purpose |
| --- | --- | --- |
| `/admin/types[/:action]` | `allAction` | `types/all` is explicitly exempted by the admin base controller; lookup data. |
| `/admin/authentication[/:action]` | `loginAction`, `loggedInDetailsAction`, `logoutAction` | Login is exempt; other actions require an admin session. |
| `/admin/uploads[/:action]` | `imageAction` | Admin session; image upload. |
| `/admin/movies`, `/admin/movies/:id[/:action]` | REST list/get/create/update/delete; `updateDetailsAction`, `releaseListAction` | Admin session; movie content. |
| `/admin/records-archive[/:id][/:action]` | REST CRUD | Admin session; records content. |
| `/admin/articles[/:id][/:action]` | REST CRUD; list accepts pagination/filter parameters; get accepts `includeSubarticles` | Admin session; articles. |
| `/admin/articles/:article_id/subarticles[/:id][/:action]` | get/create/update/delete | Admin session; nested article content. |
| `/admin/celebs`, `/admin/celebs/:id[/:action]` | REST CRUD; `birthAndDeathAction` | Admin session; celebrity profiles. |
| `/admin/banners[/:id][/:action]` | REST CRUD | Admin session; banners. |
| `/admin/do-you-know[/:id][/:action]` | REST CRUD; `activateAction` | Admin session; facts content. |
| `/admin/transactions[/:id][/:action]` | `getList` | Admin session; transaction listing. |
| `/admin/users[/:id][/:action]` | `getList`, `get` | Admin session; user inspection. |
| `/admin/admins[/:id][/:action]` | REST CRUD | Admin session; administrator management. |
| `/admin/dashboard[/:action]` | `getDataAction` | Admin session; dashboard aggregates. |
| `/admin/metatags`, `/admin/metatags/:url`, `/admin/metatags/:id[/:action]` | REST CRUD | Admin session; metadata. |
| `/admin/games`, `/admin/games/:game_type[/:id][/:action]` | game-type REST handlers; `activateAction`, `deactivateAction` | Admin session; game definitions/questions. |
| `/admin/blogs[/:id][/:action]` | REST CRUD | Admin session; blog content. |
| `/admin/subscriptions[/:id][/:action]` | `getList` | Admin session; subscription listing. |
| `/admin/poll-week-questions[/:id][/:action]` | REST CRUD | Admin session; weekly poll content. |

The route config contains broad optional action segments. The handler's method name and source controller are authoritative for whether an operation is implemented; inherited base REST methods return a 405-style error where not overridden.

## User Route Inventory

| Route | Handler methods/actions discovered | Purpose and access evidence |
| --- | --- | --- |
| `/user/types[/:action]` | `allAction` | Lookup data. |
| `/user/authentication[/:id][/:action]` | login, logged-in details, logout, register, callback, verify, forgot password, create password, profile edit, referral submit | Account workflow; guard behavior is defective/inconsistent and documented under security findings. |
| `/user/account[/:action]` | `saveImageAction` | Account image update. |
| `/user/uploads[/:action]` | `imageAction` | Upload action. |
| `/user/movies`, `/user/movies/:slug`, `/user/movies/:id`, `/user/movies/typeahead` | list, get, slug, typeahead | Public movie browse/detail/search. |
| `/user/records-archive[/:id][/:action]` | `getList` | Public records listing. |
| `/user/articles`, `/user/articles/:slug`, `/user/articles/:id[/:action]` | list, get, slug, like | Public article browse/detail/like. |
| `/user/subscriptions[/:action]` | `getList`, `create` | User subscription/purchase flow. |
| `/user/users[/:id][/:action]` | `getList` | User list handler; authorization/data contract requires review. |
| `/user/celebs`, `/user/celebs/:slug`, `/user/celebs/:id`, `/user/celebs/typeahead` | list, get, slug, typeahead | Celebrity browse/search. |
| `/user/banners[/:id][/:action]`, `/user/do-you-know[/:id][/:action]` | list/get | Public content. |
| `/user/poll-week-questions[/:id][/:action]` | get, `pollSubmitAction`, `lastWeekResultAction` | Poll participation/results. |
| `/user/blogs`, `/user/blogs/:slug`, `/user/blogs/:id[/:action]` | list, get, slug; write handlers are commented out | Public blog reads. |
| `/user/dashboard[/:action]` | home/contest/data/leaderboard/popup actions | Dashboard and leaderboards. |
| `/user/contactus[/:id][/:action]` | create | Contact submission. |
| `/user/search` | `getList` | Search. |
| `/user/products`, `/user/products/:slug`, `/user/products/:id[/:action]` | list, slug | Product catalog. |
| `/user/categories[/:id][/:action]` | list/get | Categories. |
| `/user/transactions[/:id][/:action]` | list/create | Transactions/redeem flow; external catalog/order API is called by service. |
| `/user/metatags[/:id][/:action]` | list/get | Public metadata. |
| `/user/games`, `/user/games/:game_type`, `/user/games/:game_type/:id[/:action]` | list/get, submit, rate, next, leaderboard/contest actions | Nine game types are resolved in `GameTypeController`; submit results and points are calculated in API services. |

## Selected Contracts (Code-Inferred, Not Runtime Observed)

### Login

`POST /user/authentication/login` or `POST /admin/authentication/login`; expected JSON fields are `email` and `password`. Password is checked with Laminas Bcrypt; success stores an identity in a Laminas session container. User/admin response envelopes contain a user/admin model. No token issuance was found. Runtime cookies, statuses, and all serialized fields need verification.

```json
{"email":"person@example.invalid","password":"replace_with_local_value"}
```

### Game Submission

`POST /user/games/{game_type}/{id}/submit`; game-specific payloads vary. For example, service code consumes question answers or grid characters and optional duration; scoring is performed in API services and updates status/mints tables. Do not trust client score/mints fields unless independently validated by each service. Full per-game schema is in the corresponding `Application/Service/Games/*Service.php`.

### Password Reset Contract Concern

The public web wrapper posts a bare `password` value to `/authentication/{id}/createpassword`, but the service expects parsed data with a `password` key. The public endpoint guard and reset token design also need correction/verification. Treat the UI call as a likely contract mismatch, not a verified successful flow.

## Consumer Comparison and Mismatches

- Admin wrappers cover the configured admin route families; no complete automated path/method comparison was executed.
- Public wrappers call auth, content, games, poll, rewards, products, transactions, and dashboard routes represented in the API configuration. `top3contests` maps to a slug/action route; `gamewisecontests` maps to a controller action, but runtime matching is unverified.
- Reset-password request body is likely inconsistent with service parsing as described above.
- Web clients use cookies and also expose a helper to set an Authorization header; no API bearer-token validator was found. Do not assume the header protects requests.
- Pagination uses query parameters such as `page` and `limit` in admin wrappers and data-dependent filters in public wrappers; exact defaults/maximums are table-service-specific.

For source routes and controller details see [Admin route config](../../api/ravi_bollymints-api-25c52a20b786/module/Admin/config/module.config.php), [User route config](../../api/ravi_bollymints-api-25c52a20b786/module/User/config/module.config.php), and the UI API wrapper folders linked in [REPOSITORY_STRUCTURE.md](REPOSITORY_STRUCTURE.md).