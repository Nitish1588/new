# Data Flow

All diagrams are source-derived. External calls, HTTP status mapping, and DB transactions require runtime verification.

## User Login and Session

```mermaid
sequenceDiagram
    actor Member
    participant Web as Public Next.js
    participant API as User API
    participant Auth as UserAuthenticationService
    participant DB as SQL tables
    Member->>Web: Submit email/password
    Web->>API: POST /user/authentication/login with credentials
    API->>Auth: Parse JSON and look up email
    Auth->>DB: Read users and related counts/poll/subscription
    Auth->>Auth: Verify Bcrypt hash and account status
    Auth-->>API: ResponseModel
    API->>API: Store user in Laminas session on success
    API-->>Web: JSON envelope + session cookie
    Web-->>Member: Render signed-in state
```

Logout removes the user/admin value from the session container. No bearer-token issuance/validation path was found. Cookie/CORS behavior depends on deployment configuration.

## Game Submission (API)

```mermaid
sequenceDiagram
    actor Player
    participant Web as Game page
    participant API as User API
    participant Type as GameTypeController
    participant Logic as GameTypeService / game service
    participant DB as Game and score tables
    Player->>Web: Answer game
    Web->>API: POST /user/games/{type}/{id}/submit
    API->>Type: Route, session context, game type resolution
    Type->>Logic: Pass game id, body and user id
    Logic->>DB: Load puzzle, calculate result, write status/counts
    DB-->>Logic: Persisted result
    Logic-->>API: Result and updated totals
    API-->>Web: JSON response envelope
    Web-->>Player: Show result/mints
```

Per-game scoring is not uniform. Grid and MatchPoint currently consume client-supplied correctness flags; validate server-side scoring before relying on this flow for rewards.

## Admin CRUD

```mermaid
flowchart LR
    Staff[Admin form] --> Client[Admin API wrapper]
    Client --> Route[/admin resource route/]
    Route --> Guard[Admin session guard]
    Guard --> Controller[CRUD controller]
    Controller --> Service[Domain service]
    Service --> Model[Input-filter model]
    Model --> Table[TableGateway]
    Table --> DB[(SQL database)]
    DB --> Controller
    Controller --> Client
    Client --> Staff
```

Admin content create/update passes the current admin id to services for audit fields. The declared `checkPermission()` helper has no call sites in admin controllers; authenticated non-superadmin access may therefore exceed the permission map's apparent intent.

## Purchase / Redemption

```mermaid
sequenceDiagram
    actor Member
    participant Web
    participant API as TransactionsController/Service
    participant DB
    participant Ext as External catalog/order API
    Member->>Web: Redeem product/points
    Web->>API: POST /user/transactions
    API->>DB: Read user, mints, prior redemptions, product and token settings
    API->>Ext: Fetch product / create order using configured credentials
    Ext-->>API: Provider response
    API->>DB: Record completed transaction and update related totals when successful
    API-->>Web: JSON envelope
    Web-->>Member: Show result
```

Provider verification, idempotency, transaction boundaries, and retry behavior have not been observed. No payment webhook or background worker was established in this pass.