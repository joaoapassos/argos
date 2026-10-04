# Argos

## Stack

- Go
- Chi v5
- MySQL
- database/sql
- golang-migrate

## Project rules

- Follow idiomatic Go conventions.
- Keep `cmd/` minimal.
- Application code belongs in `internal/`.
- External data sources belong in `internal/sources/`.
- HTTP handlers must not contain business logic.
- Business logic must not depend on Telegram or HTTP.
- Use `database/sql`; do not introduce an ORM without explicit approval.
- Use parameterized SQL queries.
- All schema changes must use `golang-migrate`.
- Database migrations must target MySQL.
- Never modify an already-applied migration.

## Reference

When implementation details are uncertain, prefer official documentation:

- Go: https://go.dev/doc/
- database/sql: https://pkg.go.dev/database/sql
- Chi: https://github.com/go-chi/chi
- golang-migrate: https://github.com/golang-migrate/migrate
- MySQL: https://dev.mysql.com/doc/

## Documentation

- Exported Go types, functions, methods and constants must use GoDoc-style `//` comments.
- Important internal functions should be documented when their behavior is not obvious.
- HTTP handlers must document their route and responsibility.
- Avoid comments that merely repeat the code.
- `docs/` contains high-level architecture, business rules and design decisions.
- Documentation in `docs/` should reference relevant source files when appropriate.
- Implementation details belong near the code; architectural explanations belong in `docs/`.

## Agent Skills

Use installed skills when relevant, but this AGENTS.md takes precedence
when a skill conflicts with project decisions.

## Third-party dependencies

- Do not add libraries, code, or skills without checking their license.
- Prefer Apache-2.0, MIT, BSD-2-Clause, or BSD-3-Clause dependencies.
- Record incorporated third-party code, libraries, or skills in `THIRD_PARTY_NOTICES.md` when required.
- Store required license texts in `thirdparty/licenses/`.
- Update third-party notices when adding, removing, or replacing a dependency or copied skill.
- Preserve required copyright and license notices.
- Do not introduce GPL or AGPL dependencies without explicit review.