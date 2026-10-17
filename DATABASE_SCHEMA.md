# Database Schema

## Scope and Confidence

**UNVERIFIED live schema:** no database connection was supplied or used. The API has 70 concrete TableGateway classes, 40 `CREATE TABLE` statements in SQL artifacts, legacy `ALTER TABLE` migration scripts, and model/input-filter definitions. This is not a single replayable canonical schema: historical import scripts rename and alter pre-existing tables, and some API gateway targets have no complete `CREATE TABLE` statement in the inspected SQL. Do not run these migration/import files against production during audit.

The API gateway factory converts a class such as `UsersTable` to table name `users`; `AbstractModel` and specialized models describe mapped fields and validation. Types/constraints below should be considered verified only where an actual SQL declaration supplies them. No schema is asserted for the separate legacy CodeIgniter database.

## Table Inventory

The following 70 concrete gateway targets are present. The gateway/model code makes them expected application tables, not proof they all exist in a live database.

| Domain | Tables |
| --- | --- |
| Identity and access | `admins`, `users`, `role_types`, `status_types`, `gender_types` |
| Content and media | `articles`, `sub_articles`, `article_celeb_details`, `blogs`, `blog_categories`, `blog_views_tracking`, `banners`, `records_archive`, `do_you_know`, `contactus`, `meta_tags`, `view_article_tracking`, `categories` |
| Film and people | `films`, `celebs`, `film_banner_details`, `film_celeb_details`, `film_genre_details`, `film_role_types`, `genre_types`, `certification_types`, `verdict_types` |
| Games and gameplay | `games`, `game_types`, `level_types`, `game_celeb_details`, `game_ratings`, `game_user_ratings`, `game_sponsered_types`, `game_status`, `sponsered_game_status`, `crossword_details`, `crossword_questions`, `crossword_status`, `fill_me_questions`, `fill_me_status`, `grid_details`, `grid_questions`, `grid_status`, `hi5_details`, `hi5_status`, `jumble_details`, `jumble_questions`, `jumble_status`, `match_point_details`, `match_point_questions`, `match_point_status`, `mixed_bag_details`, `mixed_bag_status`, `pix_fix_details`, `pix_fix_status`, `quiz_of_the_week_questions`, `quiz_of_the_week_status`, `games_count`, `mints_count`, `badges`, `badge_types` |
| Commerce and account activity | `products`, `subscriptions`, `transactions`, `token_settings`, `referral_users` |
| Other lookup/records | `review_details`, `poll_week_questions`, `poll_week_status` |

Additional DDL artifact: `sessions` is declared in `data/db/sessions.sql` and used by Laminas DB sessions. It is not one of the 70 application TableGateway targets.

## Verified DDL Samples

The DDL files provide types, nullability/defaults, indexes, and primary/foreign keys for tables they create. Examples:

- `grid_details`: `id` auto-increment integer PK; required `game_id`, `selected_rows`; timestamps/admin audit fields; `is_deleted` default 0; FK to `games` and `admins`.
- `grid_questions`: `id` auto-increment; `game_id`, `question_no`, `characters`, `answer`; audit/deletion fields; FK to `games` and `admins`.
- `grid_status`: `id`, `game_id`, `user_id`, question/correct/wrong counters; FKs to `games` and `users`.
- `article_celeb_details`: `id`, `article_id`, `celeb_id`, audit/deletion fields; FKs to `articles`, `celebs`, and `admins`.
- `sessions`: `id` and session payload/lifetime columns from the session SQL. Exact runtime/session settings are in local config.

Use the concrete DDL sources for every column rather than treating model properties as a canonical physical schema: [new migrations](../../api/ravi_bollymints-api-25c52a20b786/data/db/migrations_new/), [legacy migration/import SQL](../../api/ravi_bollymints-api-25c52a20b786/data/db/migration_from_old_db/), [session DDL](../../api/ravi_bollymints-api-25c52a20b786/data/db/sessions.sql).

## Keys and Relationships

The migration corpus contains 149 FK clauses, including repeated definitions. Declared relationships include content and film joins, game definitions/statuses, admin audit ownership, and user/game activity. Examples include `users.role_type_id -> role_types.id`, `users.status_type_id -> status_types.id`, `games.game_type_id -> game_types.id`, `games.level_type_id -> level_types.id`, `sub_articles.article_id -> articles.id`, `film_celeb_details` to films/celebs/film roles, and game status `game_id`/`user_id` to games/users.

The SQL files do not consistently include unique constraints for natural identifiers. Some table names/columns only appear in `ALTER TABLE` scripts for the historical database. Consult [ER_DIAGRAM.md](ER_DIAGRAM.md) for relationship evidence and confidence boundaries.

## Models, APIs, Admin, Lifecycle

- Models and validation filters: `api/.../module/Application/src/Model/*.php`.
- Query and write behavior: `api/.../module/Application/src/Table/*.php`; generic operations are in `AbstractTable` and domain joins in subclasses.
- API access: admin routes/controllers commonly read/write content, users, games, transactions, and metadata; public routes read content and game data and write account, contact, poll, purchase, and game results. The exact route-to-table map is summarized in [API_REFERENCE.md](API_REFERENCE.md).
- Admin UI pages are in `admin/.../pages/`; public consumers are in `web/.../pages/`. Admin/user client wrappers are in each module's `src/api/` or `src/definitions/apis/`.
- Common model deletion fields (`is_deleted`, `deleted_on`, `deleted_by`) show soft-delete support in some entities. Actual delete behavior varies by Table subclass; do not assume all tables cascade or soft-delete.
- **UNVERIFIED:** retention, physical deletion, production indexes/unique constraints, seed data, exact foreign-key state, and whether legacy game data is migrated into this database.

## Data Quality Notes

Migration history is split between legacy schema alterations and newer additions. This makes migration order/environment critical. Foreign-key coverage is uneven: some newer game/content tables declare constraints; old entity definitions are often ALTER-only, so missing constraints cannot be distinguished from omitted baseline schema. Review canonical production DDL before any schema change. Index performance, orphan records, N+1 patterns, and cascade behavior need a read-only live-schema/query review.