# Tasks

## 1. Project scaffolding

- [ ] 1.1 Create `/backend/app` and `/frontend/{css,js}` directories plus `backend/requirements.txt` pinning `fastapi`, `uvicorn[standard]`, `pytest`, and `httpx`; verify `pip install -r backend/requirements.txt` succeeds in a fresh virtual environment.
- [ ] 1.2 Add `.gitignore` entries for `backend/data/`, `__pycache__/`, `.venv/`, and `*.pyc`; verify `git status --short` does not list a created database file or bytecode.
- [ ] 1.3 Create `backend/app/main.py` with a bare FastAPI app and `GET /api/health` returning `{"status": "ok"}`; verify `uvicorn app.main:app` (run from `backend/`) starts and `curl http://127.0.0.1:8000/api/health` returns the JSON.

## 2. Persistence layer

- [ ] 2.1 Implement `backend/app/db.py` with a `TODO_DB_PATH`-aware connection factory and idempotent `todos` table creation per design.md; verify calling `init_db()` twice against a temp path creates the file once and raises no error.
- [ ] 2.2 Implement `backend/app/repository.py` with `list_todos`, `get_todo`, `insert_todo`, `update_todo`, and `delete_todo` that return domain `Todo` objects and never expose `sqlite3.Row`; verify with repository-level tests using a temp database.
- [ ] 2.3 Add `backend/tests/test_repository.py` covering insert/list ordering (newest first), get-miss returning `None`, partial update, delete, and reopening a second connection against the same file to confirm items remain; verify `pytest backend/tests/test_repository.py` passes.

## 3. ToDo REST API

- [ ] 3.1 Add Pydantic models in `backend/app/models.py` (`TodoCreate`, `TodoUpdate`, `TodoRead`) with trimmed 1-200 char titles, 2000 char descriptions, strict boolean `completed`, and `extra="forbid"`; verify model tests reject empty/whitespace titles and non-boolean `completed`.
- [ ] 3.2 Implement `backend/app/routes/todos.py` with `GET/POST /api/todos` and `GET/PATCH/DELETE /api/todos/{id}` wired to the repository through the `get_connection` dependency; verify success statuses (200/201/204) and returned item shape via `TestClient`.
- [ ] 3.3 Map validation failures to 422 with a field-identifying body and unknown ids to 404 with `{"detail": "ToDo item not found"}`, on both `PATCH` and `DELETE`; verify each path with tests.
- [ ] 3.4 Add `backend/tests/test_api.py` covering create→list→edit→toggle→delete end to end, empty-title 422, non-boolean `completed` 422, unknown-id 404 on update and delete, `created_at` unchanged by edits, and ISO-8601 UTC timestamp format; verify `pytest backend/tests/test_api.py` passes.
- [ ] 3.5 Confirm the generated OpenAPI docs list every ToDo endpoint with request and response schemas; verify by fetching `/openapi.json` in a test and asserting the five paths and methods are present.

## 4. Public pages

- [ ] 4.1 Build `frontend/index.html`, `about.html`, `portfolio.html`, and `contact.html` with a shared header/nav (Home, About, Portfolio, Contact), `aria-current="page"` on the active link, and no link to the ToDo page; verify all four render in the browser and each nav link resolves.
- [ ] 4.2 Fill the About page with distinct biography, education, skills, and interests sections, and the Contact page with an `mailto:` email link plus labelled optional channels rendered with `rel="noopener"` and `target="_blank"` for external profiles; verify the Contact page contains no form or input elements.
- [ ] 4.3 Add `frontend/js/projects.js` with the project array and render it in `portfolio.html` — each technology as its own list item, a link row only when `url` exists, and an empty-state message when the array is empty; verify by temporarily removing all entries that the empty state shows and no broken link renders.
- [ ] 4.4 Write `frontend/css/styles.css` with custom properties, a wrapping nav, and a single-column max-width layout; verify at a 400px viewport that no page scrolls horizontally and text stays readable.

## 5. ToDo page and client

- [ ] 5.1 Add `frontend/todo.html` and `frontend/js/api.js` (`listTodos`, `createTodo`, `updateTodo`, `deleteTodo`) that throw errors carrying the server's `detail` message for non-2xx responses; verify each function succeeds against the running API and that a forced 404 surfaces as a thrown error.
- [ ] 5.2 Implement `frontend/js/todo.js` rendering all items newest first with title, description, creation date, and an unambiguous completion checkbox, plus an inline create form and inline edit form with client-side non-empty title checks; verify the rendered list matches `GET /api/todos` after each action.
- [ ] 5.3 Wire completion toggling (`PATCH {completed}`), deletion behind a confirmation prompt, and a visible error message that leaves the displayed list consistent with the server on failure; verify cancelling a deletion keeps the item and that a stopped backend produces an on-page error rather than a silent no-op.
- [ ] 5.4 Verify the empty-list state and the first-start behavior with no database file present — delete `backend/data/todos.db`, start the app, and confirm the page shows an empty-state message while the create form still works.

## 6. Integration and documentation

- [ ] 6.1 Mount `frontend/` with `StaticFiles(html=True)` after the API routers so `/` serves the homepage and `/api/*` still resolves; verify `/`, `/about.html`, `/portfolio.html`, `/contact.html`, `/todo.html`, and `/api/todos` all return 200 from `TestClient` and add these assertions to `backend/tests/test_api.py`.
- [ ] 6.2 Run the manual end-to-end pass in a browser: create items, restart the server, confirm items and their completion state persist, then edit, complete, uncomplete, and delete one; record the outcome in the README verification section.
- [ ] 6.3 Write `README.md` run and test instructions (setup, `uvicorn`, `pytest`, URLs, and the project layout) and document the known limitation that the ToDo page is private by convention only, with no authentication; verify the documented commands work verbatim from a clean shell.
- [ ] 6.4 Run the full suite with `pytest backend/tests` and confirm every test passes with no skipped tests.

## Workflow follow-up

- Implement this change with the `openspec-apply-with-github` workflow, tracking tasks against the GitHub issue.
- Archive the change with `openspec-archive-with-github` once review requirements are satisfied, then verify the archived change and the updated main specs.
