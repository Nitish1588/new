# Project Analysis

**Audit status: partial static audit.** This is an evidence-based first pass over the extracted source trees, not a claim of runtime or production verification. The database is unavailable, frontend/PHP dependencies are not installed, and the 24,602-file `game/` archive includes extensive bundled libraries/assets. See [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md) and [AUDIT_CHECKLIST.md](AUDIT_CHECKLIST.md).

## Executive Summary

Bollymints is a Bollywood content and casual-game platform. The workspace contains four separately rooted applications: two Next.js frontends, a Laminas MVC JSON API, and an older CodeIgniter PHP site/game implementation. The code indicates content management, user accounts, game play and points, subscription/product redemption, media uploads, and admin management. It does not prove that all four applications are deployed together or share one database.

The most urgent verified concern is account mutation through API authentication/profile routes: the shared user controller's guard does not enforce authentication on most paths, and profile/password services use request-provided identifiers without verifying the current password. Additional high-risk findings include hardcoded credentials in source, TLS certificate verification disabled in the public frontend API client, and SQL built from a POST value in the legacy game controller. Details and caveats are in [SECURITY_AUDIT.md](SECURITY_AUDIT.md).

## Modules

| Module | Detected technology | Root | Responsibility indicated by source |
| --- | --- | --- | --- |
| Admin | Next.js/React, Material UI, Axios | `admin/ravi_bollymints-admin-9df7f773dd7c/` | Separate admin UI for CMS, users, games, billing records, and configuration-like content. |
| API | PHP, Laminas MVC, Laminas DB, MySQL-compatible SQL | `api/ravi_bollymints-api-25c52a20b786/` | JSON API, services, table gateways, authentication, game scoring, purchases, uploads. |
| Game | PHP, CodeIgniter-style MVC | `game/ravi_bollymints-bollymints-6a962ebafcfa/` | Legacy site/admin/game flows, direct legacy database access, and bundled assets/libraries. |
| Web | Next.js 10.2, React 17, Redux Toolkit, Axios | `web/ravi_bollymints-web-97661d85ac77/` | Public content, user account, game, membership, points/rewards pages. |

## Documentation Index

- [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md)
- [REPOSITORY_STRUCTURE.md](REPOSITORY_STRUCTURE.md)
- [ARCHITECTURE.md](ARCHITECTURE.md)
- [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md)
- [ER_DIAGRAM.md](ER_DIAGRAM.md)
- [API_REFERENCE.md](API_REFERENCE.md)
- [FEATURE_INVENTORY.md](FEATURE_INVENTORY.md)
- [CONTROL_MAP.md](CONTROL_MAP.md)
- [DATA_FLOW.md](DATA_FLOW.md)
- [ADMIN_GUIDE.md](ADMIN_GUIDE.md)
- [GAME_MODULE.md](GAME_MODULE.md)
- [SECURITY_AUDIT.md](SECURITY_AUDIT.md)
- [TESTING_AND_BUILD.md](TESTING_AND_BUILD.md)
- [TECHNICAL_DEBT.md](TECHNICAL_DEBT.md)
- [DEVELOPER_ONBOARDING.md](DEVELOPER_ONBOARDING.md)
- [TRACEABILITY_MATRIX.md](TRACEABILITY_MATRIX.md)
- [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)
- [CHANGE_IMPACT_GUIDE.md](CHANGE_IMPACT_GUIDE.md)
- [AUDIT_CHECKLIST.md](AUDIT_CHECKLIST.md)

## Evidence and Scope

Claims are tagged **CONFIRMED**, **INFERRED**, or **UNVERIFIED**. Active route patterns and handlers were statically counted from the API route configs/controllers; the count is not a normalized HTTP endpoint count because route templates are action-dispatched and optional. No external credentials or secret values are reproduced here. Source paths are linked relative to this workspace.