# Design

## Context

See `proposal.md` - Why. The repository has no application code yet; it contains only `openspec/` and a README. Project constraints from `.agents/skills/AGENTS.md` and `openspec/config.yaml` fix the stack: Python/FastAPI backend, HTML/JavaScript frontend, SQLite storage, REST/JSON between them, business logic and database access in the backend, dependency injection in the backend, and a `/backend` + `/frontend` layout. This is an educational single-user application, so the design favors explicit, readable code over framework machinery.

## Goals / Non-Goals

**Goals:**

- One runnable process for local development: the FastAPI app serves both the REST API and the static frontend, so the frontend and API share an origin.
- A small, well-separated backend: request/response models, a ToDo service holding business rules, and a repository owning all SQL against SQLite.
- A testable API contract with automated coverage for the CRUD, validation, not-found, and persistence behaviors in the specs.

**Non-Goals:**

- Authentication, sessions, users, or roles. The ToDo page is private by convention (direct URL, absent from public navigation), not by enforcement.
- Content management for the public pages; site text and project entries are static files edited in the repository.
- Framework-based frontend, bundler, npm toolchain, or client-side routing.
- Pagination, filtering, search, due dates, priorities, tags, or attachments for ToDo items.
- Production deployment topology beyond running the app under an ASGI server.

## Decisions

### 1. FastAPI serves the static frontend from `/frontend`

`app.mount("/", StaticFiles(directory=FRONTEND_DIR, html=True))` is registered after the API routers, so `/api/...` routes win and everything else resolves to files in `/frontend`.

- **Why:** a single origin removes CORS configuration, avoids a second dev server, and keeps `fetch("/api/todos")` valid in every environment.
- **Alternative considered:** separate static file server (e.g. `python -m http.server`) with permissive CORS. Rejected: extra moving part and a CORS surface that adds nothing for a single-page-per-file site.
- **Consequence:** no server-side templating. Shared markup (header/nav/footer) is duplicated across pages; acceptable for five pages and keeps the HTML directly readable to students.

### 2. Persistence: stdlib `sqlite3` behind a repository module

`backend/app/db.py` owns the connection factory and schema creation; `backend/app/repository.py` holds every SQL statement (`list_todos`, `get_todo`, `insert_todo`, `update_todo`, `delete_todo`). Rows are converted to domain `Todo` objects inside the repository, so SQL and `sqlite3.Row` never leak into services or routes.

- **Why:** `sqlite3` is in the standard library, has no migration tooling to learn, and keeps the mapping visible. It also satisfies "business logic in the backend" without ORM magic.
- **Alternative considered:** SQLAlchemy/SQLModel. Rejected for the initial version: more dependencies and concepts than a five-column table needs; the repository boundary means it can be swapped later without touching the API.
- Schema, created idempotently at startup (`CREATE TABLE IF NOT EXISTS todos (...)`):

  | column       | type    | notes                                        |
  | ------------ | ------- | -------------------------------------------- |
  | `id`         | INTEGER | primary key, autoincrement                    |
  | `title`      | TEXT    | NOT NULL, trimmed, non-empty                  |
  | `description`| TEXT    | NOT NULL, default `''`                        |
  | `completed`  | INTEGER | NOT NULL, default 0, exposed as JSON boolean  |
  | `created_at` | TEXT    | NOT NULL, ISO 8601 UTC timestamp              |

- The database file lives at `backend/data/todos.db` and is overridable with the `TODO_DB_PATH` environment variable so tests and demos can point at a temporary file. `backend/data/` is git-ignored.

### 3. Connection and transaction handling

Each request opens its own short-lived connection through a FastAPI dependency (`get_connection`), which yields it and closes it in a `finally` block; `PRAGMA foreign_keys = ON` and `row_factory = sqlite3.Row` are set on open, and writes commit explicitly.

- **Why:** avoids sharing one connection across threads, and demonstrates dependency injection as the project rules require.
- **Trade-off:** one connection per request has negligible cost at this scale and is simpler than a connection pool.

### 4. API shape: resource-oriented `/api/todos`

| Method | Path              | Body                                   | Success            |
| ------ | ----------------- | -------------------------------------- | ------------------ |
| GET    | `/api/todos`      | -                                      | 200, array of items |
| POST   | `/api/todos`      | `{title, description?}` (no status)    | 201, created item   |
| GET    | `/api/todos/{id}` | -                                      | 200, item           |
| PATCH  | `/api/todos/{id}` | any of `{title, description, completed}`| 200, updated item   |
| DELETE | `/api/todos/{id}` | -                                      | 204, empty body     |

Item representation: `{"id": 1, "title": "...", "description": "...", "completed": false, "created_at": "2026-01-31T09:15:00Z"}`.

- **Why PATCH for every edit:** editing text and toggling completion are the same operation from the UI's point of view; a partial update lets the completion checkbox send only `completed` and the edit form send only `title`/`description`.
- **Alternatives considered:** `PUT /api/todos/{id}` with a full body (forces the client to resend fields it is not changing) and `POST /api/todos/{id}/toggle` (less RESTful, duplicates read-modify-write on the server). Rejected.
- **Ordering:** `GET /api/todos` returns newest first (`ORDER BY created_at DESC, id DESC`) so the sort order is deterministic when timestamps tie.
- Pydantic models (`TodoCreate`, `TodoUpdate`, `TodoRead`) enforce the contract: title trimmed and 1-200 characters, description at most 2000 characters, `completed` strictly boolean, `extra="forbid"` so misspelled or unknown fields fail loudly.

### 5. Error semantics

- Invalid payload → `422` with FastAPI's default `{"detail": [{"loc": [...], "msg": ...}]}` body; the message names the offending field.
- Unknown or malformed id → `404` with `{"detail": "ToDo item not found"}`. Unknown id on `PATCH` and `DELETE` returns `404` (not upsert, not silent success).
- Storage failure → `500` with `{"detail": "..."}`; the service logs the exception.
- **Why:** the frontend needs a stable contract to display the "title is required" and generic failure messages required by the `todo-list` spec, while relying on standard FastAPI validation rather than hand-writing checks.

### 6. Frontend: five static pages plus shared CSS/JS

- `frontend/index.html`, `about.html`, `portfolio.html`, `contact.html` - public pages sharing `css/styles.css` and a hand-written header/nav block; the current page link carries `aria-current="page"`.
- `frontend/todo.html` - the private page. It is deliberately absent from the public navigation and from any public page's markup.
- `frontend/js/api.js` - a thin `fetch` wrapper exporting `listTodos`, `createTodo`, `updateTodo`, `deleteTodo`, turning non-2xx responses into thrown errors carrying the server's `detail` message.
- `frontend/js/todo.js` - renders the list, wires the create form, inline edit form, completion checkbox, and delete confirmation (`window.confirm`), and re-reads the server's response (or refetches the list) after every mutation so the displayed state always matches storage.
- `frontend/js/projects.js` - the project data as an array of objects (`title`, `description`, `technologies`, optional `url`); `portfolio.html` renders it, showing each technology as its own `<li>` and omitting the link row when `url` is absent.
- **Why:** a data array instead of hand-written HTML for projects keeps the "optional link" and "technologies as separate values" requirements in one place and makes adding a project a one-object edit.
- **Why no build step:** plain `<script>` files loaded with `type="module"` need no transpiler, and browsers resolve them directly.
- **XSS safety:** item and project text is inserted with `textContent` / `document.createElement`, never `innerHTML`.

### 7. Styling and responsiveness

One stylesheet with CSS custom properties for color/type scale, a flexbox nav that wraps below 600px, and a single-column layout with `max-width` so text remains readable at 400px width. No CSS framework.

- **Why:** no dependencies, and the responsive behavior in the `student-profile` spec is achievable with a few media queries.

### 8. Testing

`pytest` with FastAPI's `TestClient`, a per-test temporary database injected by overriding the `get_connection` dependency and setting `TODO_DB_PATH`. Tests cover: create/list/edit/toggle/delete, empty-title rejection, unknown-id 404s, ordering, and the restart-persistence scenario (reopening the app against the same temp file). A small smoke test asserts each public HTML file is served and the navigation contains no ToDo link.

## Risks / Trade-offs

- **Private ToDo page is unauthenticated** → Anyone who knows or guesses `/todo.html` can read and change items. Accepted for the initial educational scope; documented in the README as a known limitation, and the architecture (single service, single-user data) leaves room to add auth in a later change.
- **Duplicated navigation markup across pages** → Drift between pages if the nav changes. Mitigated by keeping the nav block identical and asserting its links in a test; a shared include can be introduced if the site grows.
- **`sqlite3` writes are serialized** → Concurrent writers can hit `database is locked`. Mitigated by a short connection lifetime, small transactions, and this being a single-user app; a `timeout` is set on connection open.
- **Hand-rolled schema creation, no migrations** → Future column changes need manual work. Accepted for v1; the schema lives in one function in `db.py`, so a migration step can be added when the first schema change arrives.
- **Static project content in JS** → Editing projects means editing code, not a CMS. Accepted as a non-goal; the shape is intentionally close to what an API would return if one is added later.
- **Timestamps stored as ISO text** → Sorting lexicographically is correct only for consistent UTC formatting with a fixed precision. Mitigated by always emitting `YYYY-MM-DDTHH:MM:SSZ` through one helper function.

## Migration Plan

Greenfield: no data or consumers exist yet.

1. Add `backend/` (app, repository, schema creation) and `frontend/` files, plus `.gitignore` entries for `backend/data/` and `__pycache__/`.
2. Add `backend/requirements.txt` and a short `README.md` run section (`uvicorn app.main:app --reload` from `/backend`, then open `http://127.0.0.1:8000/`).
3. Verify the four public pages and the ToDo flow manually in the browser, plus the API via `/docs`.
4. Rollback: delete the new directories and the generated database file; nothing else is affected.

## Open Questions

None blocking. Two items can be settled during implementation without changing the specs: the actual student biography, portfolio, and contact content (placeholder content will be used until real text is supplied), and whether the app is served with `--reload` only locally or also packaged behind a plain `uvicorn` command.
