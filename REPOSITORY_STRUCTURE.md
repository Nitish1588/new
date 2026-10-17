# Repository Structure

The workspace contains extracted ZIP roots, not one conventional repository root. Paths below include the archive's nested project directory.

```text
docs/project-analysis/                 audit output
admin/ravi_bollymints-admin-.../       Next.js admin app
  pages/                               page-router routes
  src/api/                             Axios endpoint wrappers
  src/components/                      admin UI and forms
  utils/                               shared constants/helpers
  config.js, config-local.js.dist      API/image base URL selection
api/ravi_bollymints-api-.../           Laminas MVC app
  public/index.php                     HTTP bootstrap
  config/                              module and local config
  module/Application/                  shared controllers, services, models, tables
  module/Admin/                        admin API controllers/services/routes
  module/User/                         user API controllers/services/routes
  data/db/                             legacy and new SQL migration artifacts
game/ravi_bollymints-bollymints-.../   CodeIgniter legacy app/site
  index.php                            HTTP entry point
  application/config/                  routes, DB, session, app config
  application/controllers/             public/admin/game request handlers
  application/models/                  legacy DB access/business methods
  application/views/                   PHP-rendered UI
  application/libraries/               mostly bundled third-party libraries
  system/, assets/, upload_*/           framework, static assets, user content
web/ravi_bollymints-web-.../            public Next.js app
  pages/                               public page routes
  src/definitions/apis/                 Axios endpoint wrappers and client
  src/components/, src/utils/           UI, state and helpers
  public/, amplify.yml                  public assets/deploy config
```

## Entry Points and Manifests

| Module | Entry/config | Build/test setup |
| --- | --- | --- |
| Admin | `admin/.../pages/_app.js`, `next.config.js`, `package.json` | `npm run dev`, `npm run build`, `npm run lint`; no test script. |
| API | `api/.../public/index.php`, `config/application.config.php`, `module/*/config/module.config.php` | Composer scripts include `serve`, `test`, `cs-check`, `static-analysis`. |
| Game | `game/.../index.php`, `application/config/{autoload,routes,database,config}.php` | Apache/PHP application; no application test suite found. |
| Web | `web/.../pages/_app.jsx`, `next.config.js`, `src/config.js`, `package.json` | `npm run dev`, `npm run build`, `npm run lint`; no test script. |

## Ownership and Generated/Third-Party Content

`game/application/libraries/` contains 6,074 PHP files, including vendored Google/Facebook/PHPMailer code; `system/`, CKEditor, and large asset/upload trees are not first-party feature code. Exclude these from per-file feature inventory unless a specific integration is being traced. The extracted module totals are admin 196, API 399, game 24,602, and web 357 files; these are archive file counts, not owned-source counts.

Key source references: [API Composer manifest](../../api/ravi_bollymints-api-25c52a20b786/composer.json), [public package manifest](../../web/ravi_bollymints-web-97661d85ac77/package.json), [admin package manifest](../../admin/ravi_bollymints-admin-9df7f773dd7c/package.json), [legacy routes](../../game/ravi_bollymints-bollymints-6a962ebafcfa/application/config/routes.php).