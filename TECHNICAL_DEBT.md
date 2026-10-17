# Technical Debt

Priorities are recommendations, not code changes. Findings are based on static evidence and should be verified in the deployed environment.

| Priority | Issue | Evidence / impact | Safe remediation plan |
| --- | --- | --- | --- |
| P0 | User account authorization/reset flow is unsafe | User base-controller guard and profile/reset services allow sensitive account mutation without a proven authenticated identity or reset token. See SEC-01. | Patch authorization first, add regression tests, inspect session deployment, then invalidate affected reset links/sessions if needed. |
| P0 | Source-embedded credentials | API auth and legacy social login source contain integration credentials. See SEC-02. | Rotate/revoke, audit git/archive distribution, move secrets to a vault, and add automated scanning. |
| P1 | API TLS verification disabled | Public web Axios client sets Node TLS verification off. See SEC-03. | Remove override and test server-side calls with trusted/untrusted certs. |
| P1 | Legacy SQL injection and duplicate persistence | Legacy `Welcome` submit methods interpolate POST/session values into SQL; scoring and tables overlap modern API. | Inventory active route usage, bind all raw SQL, establish canonical writer before migration or deprecation. |
| P1 | Client-controlled game correctness | Grid/MatchPoint use client `is_correct` payload values to calculate earned points. | Recompute on server from answers; add integration tests and reconcile balances if production use is confirmed. |
| P1 | Admin role permissions are not enforced at action sites | Permission map/helper exists but concrete controllers do not call it. | Centralize policies and test role/operation matrix on direct API requests. |
| P1 | Schema source is fragmented | 70 gateway targets vs migration files that include old ALTER-only schema and newer CREATE scripts; no canonical baseline. | Capture sanitized production DDL read-only, establish migration lineage, add schema diff/restore tests. |
| P2 | Two game stacks and likely duplicated rules | Next/Laminas API and legacy CodeIgniter both implement game flows; active deployment and shared data are unknown. | Establish traffic/source-of-truth; document a parity matrix before consolidating or deprecating either side. |
| P2 | Dynamic and broad route templates | 67 active route patterns use optional segments/action dispatch; frontend wrappers are not exhaustively contract-tested. | Generate an OpenAPI contract from explicit route definitions and add route/client parity tests. |
| P2 | Weak migration of responsibilities into generic service/table layers | Generic `AbstractTable`, model filters, and services centralize varied behaviors; correctness depends on entity-specific overrides. | Keep changes domain-scoped; add contract tests before extracting common behavior. |
| P2 | Dependency drift | Admin manifest uses `latest` for important packages; older Next/React packages and unbounded majors create nondeterminism. | Pin supported versions, review lockfiles and runtime compatibility, then upgrade with CI coverage. |
| P3 | Sparse tests and onboarding docs | Only a skeleton API controller test was found; UI and legacy app lack owned suites. | Build an incremental test pyramid around account security, CRUD, gameplay, and commerce. |

Do not combine security remediation, schema changes, and legacy decommissioning into one rollout. Preserve behavior with tests and staged migration/rollback plans.