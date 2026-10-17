# Control Map

| Behavior or setting | Authoritative source | Related files | Database/config dependency | Affected features |
| --- | --- | --- | --- | --- |
| API module registration and startup | `api/.../public/index.php`, `config/application.config.php` | `module/*/config/module.config.php`, `Module.php` | PHP config and Composer autoload | All API routes. |
| Public/admin API URL | `web/.../src/config.js`, `admin/.../config.js` | local config dist files, Axios client | Build-time JS config; checked-in defaults use test API host | All frontend API calls. |
| API route prefixes and dispatch | `module/User|Admin/config/module.config.php` | Controller factories and actions | API deployed mount prefix is external | Frontend/API contract. |
| API session storage | `Application/Module.php` starts a Laminas session DB handler | session config template, `sessions.sql`, controller base classes | `sessions` table and cookie settings | User/admin identity. |
| User login/profile access | `User/Controller/AbstractUserController.php` and `AuthenticationController.php` | `UserAuthenticationService.php`, UsersTable, frontend auth wrappers | Laminas `User` session container; user account rows | Registration, login, password reset, game submissions. Current guard is defective; see security report. |
| Admin login and role check | `AdminAuthenticationService.php`, `AbstractAdminController.php` | admin auth UI/API | Laminas `Admin` session container, `admins` and role data | Admin access. Fine-grained permission map is declared but no callers were found. |
| Model validation | each `Application/Model/*.php::getInputFilter()` | `AbstractModel`, `AbstractGameStatusModel` | Per-model filters; physical DB rules may differ | CRUD and gameplay. Validation must be inspected per model. |
| Persistence and soft deletion | `Application/Table/AbstractTable.php` and domain Table subclasses | corresponding models/services | SQL table names derived from `*Table` class names; `is_deleted`/audit columns in some models | Content, user, game, commerce. Delete semantics vary by subclass. |
| Game type selection | `User/Controller/Games/GameTypeController.php::getGameTypeId()` | `GameTypes` constants, `GameTypeService` | `game_types`, `games` | Nine API game types. |
| Game scoring and mint totals | `Application/Service/Games/GameTypeService.php` and per-game service classes | `GamesUtil`, status/count tables | game level/type constants, `mints_count`, status tables | Game results, badges, rewards. Some handlers trust client `is_correct`. |
| Game auth exception | `User/Controller/AbstractUserController.php` | `GameTypeController::submitAction()` | `Game` cookie value and user session | Guest game submit. Condition is unusual and should not be relied on as auth. |
| File storage | `Application/Service/AbstractService.php` upload helpers, `Application/Util/S3Upload.php`, `ImagePath.php` | upload controllers/services, frontend upload wrappers | S3-related local API config values; bucket/path rules | Profile/content/game images. |
| Email and verification URLs | `UserAuthenticationService.php` | local API config; SMTP implementation in service | website URL plus SMTP settings | Registration and password reset. Source currently contains sensitive credentials; rotate and move to secret config. |
| External catalog/order API | `TransactionsService.php` | transaction controller, subscriptions/products UI | `qc_url`, client key/secret and account config in local API config | Redeem/purchase transaction. Provider behavior unverified. |
| Legacy app routes/landing | `game/application/config/routes.php` | `Welcome`, `AdminBollyMints` | CodeIgniter config and DB config | Legacy public content, admin, and games. |
| Legacy game score/persistence | `game/application/controllers/Welcome.php` | `AdminModel`, `conn.php`, legacy views | Direct DB tables and PHP session | Legacy games and mints. Logic overlaps API but database sharing is unknown. |
| Deployment | Amplify YAML, API Docker/Vagrant files, Apache `.htaccess` | Next config and PHP entrypoints | infrastructure-owned values not present in source | Build and release. Production source of truth unknown. |

There is no evidence that an admin UI setting overrides environment/config values. Treat admin-editable content as database-backed content, not infrastructure configuration.