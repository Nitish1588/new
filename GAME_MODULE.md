# Game Module

## Two Implementations

The source contains two distinct game paths; do not conflate them:

1. **Current-looking web/API path:** Next.js pages in `web/.../pages/games/` call `/user/games/...`; Laminas `GameTypeController` resolves nine game types and delegates to `GameTypeService` plus per-game services.
2. **Legacy CodeIgniter path:** `game/.../application/config/routes.php` maps browser routes to `Welcome` and `AdminBollyMints`; PHP views render games and some submissions use direct MySQLi operations. Whether this path is still deployed or shares the API database is UNVERIFIED.

## API Game Lifecycle

Game listing/details are served by `GamesService`/`GameTypeService`; game type names map to constants for Quiz of the Week, Crossword, Jumble, Hi5, MatchPoint, FillMe, MixedBag, Grid, and PixFix. `submitAction()` routes to `GameTypeService::submitGame()`, which selects a per-game submission handler, calculates mints, applies bonus/badge rules, checks prior status, and writes game status/count totals. Services retrieve protected answers and compare them for several game types.

```mermaid
flowchart TD
    Page[Next.js game route] --> Client[GamesApi wrapper]
    Client --> Route[/user/games/{type}/{id}/submit/]
    Route --> Type[GameTypeController]
    Type --> Dispatcher[GameTypeService dispatcher]
    Dispatcher --> PerGame[Game-specific scoring service]
    PerGame --> Status[(game status/detail tables)]
    PerGame --> Counts[(mints_count / games_count)]
    Status --> Reply[ResponseModel]
    Counts --> Reply
    Reply --> Page
```

## Rules, State, and Trust Boundary

- Points/difficulty constants and double-score/badge logic are in API constants/utilities and `GameTypeService`, not wholly in the UI.
- The API persists game status, user counts and mints for supported game types. Schema is migration-derived; see [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md).
- **High integrity concern:** Grid/MatchPoint code reads submitted `questions[*].is_correct` flags to count correct answers. A caller can potentially forge those booleans unless another validation path exists; no such independent answer check was established in the service snippets reviewed. See finding SEC-06.
- Hi5 and several string-answer games compare submitted answers to server-loaded expected answers. Other game types should be reviewed individually before asserting anti-cheat guarantees.
- No comprehensive client anti-cheat mechanism or authoritative server session/gameplay event stream was found.

## Legacy Runtime

The CodeIgniter app has PHP routes for quiz, crossword, jumble, hi5, match-point, fill-me and other pages; controller methods render PHP views and process submissions. Some handlers use DB APIs and other game submit paths issue raw SQL. Legacy persistence, table definitions, and use of session `user_id` differ from the Laminas path. Avoid changing a game rule until determining which site path is live and reconciling both implementations.

## Assets and Build

The public game UI is bundled into the Next app and uses React game dependencies (including a crossword package). Legacy code uses PHP views, uploaded image directories, CKEditor and bundled libraries. No dedicated game test suite or separate game build pipeline was found. Browser/runtime behavior, asset origin, and production routing remain unverified.