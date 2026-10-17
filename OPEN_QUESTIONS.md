# Open Questions and Verification Gaps

This audit is explicitly partial. The extracted source gives substantial static evidence, but it does not satisfy a full runtime/database/infrastructure audit.

## Environment and Runtime

- Is `game/` deployed and receiving traffic, or is the Next.js + Laminas path authoritative? Do the two game systems write to the same DB?
- What production API mount/reverse-proxy creates the `/api/public` prefix? Are the checked-in test API defaults intended only for staging?
- Which PHP/Node versions and dependency lockfile resolutions are deployed? Do old CodeIgniter and PHP packages run on supported production versions?
- Which production CORS origins, cookie domain/Secure/SameSite flags, TLS endpoints, and session persistence settings are active?
- Are S3, SMTP, social identity, and external commerce integrations still active? Which credentials must be rotated, and what is the provider's idempotency contract?
- Are cron/scheduled tasks, external workers, webhooks, or observability services deployed outside the source archive?

## Database

- What is the current sanitized production DDL, including actual column types, defaults, indexes, unique keys, FK constraints, and engine/collation?
- Which migration order is authoritative? The SQL tree mixes old-database ALTER scripts and newer CREATE scripts; schema baseline and migration execution history were not available.
- Which legacy tables are used by `game/conn.php` and `application/config/database.php`, and are they the same database as Laminas `db`?
- What are exact deletion/retention rules, transaction boundaries, and data ownership/cascade semantics for users, points, subscriptions, transactions, uploaded media, and game statuses?

## API and Features

- A normalized endpoint-by-endpoint matrix of HTTP verbs, payload fields, validation, status codes, pagination and consumers remains to be generated. The current API reference catalogs route families and handler methods, but optional route templates require a full controller/service pass.
- Which endpoints intentionally permit anonymous reads/writes? The user route guard is unusual; confirm expected guest gameplay and protect account operations independently.
- What is the intended password-reset token lifecycle, password policy, email verification expiry, and social-login verification model?
- Which game types are score-critical, which verify answers server-side, and which allow guests? Validate Grid/MatchPoint payload trust and repeated submission behavior.
- What are the exact external commerce response/status mappings, retries, duplicate-order protections, and reconciliation process?
- Are API response HTTP status codes intentionally always default JSON status while numeric application codes are only in the body?

## Audit Coverage and Validation

- Full per-field schemas and per-endpoint request/response schemas were not extracted for all 70 gateway classes and 139 handler methods.
- Individual services, all model validators, all frontend API call sites, all legacy controllers/views, and all 24,602 archive files were not exhaustively reviewed. Bundled libraries/assets were grouped.
- No dependencies, DB connection, browser session, tests, build, lint, or runtime requests were executed; dependency directories are absent.
- No production infrastructure, admin user, database, logs, or stakeholders were available for confirming deployment and intended behavior.
- Mermaid blocks received editor diagnostics only; no dedicated Mermaid parser was available.