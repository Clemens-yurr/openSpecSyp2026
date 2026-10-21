# Proposal

## Why

The project currently contains no application code, only OpenSpec configuration. This change defines the first working version of an educational project: a personal website for a student plus a small private ToDo list. It establishes the public pages a visitor can read and the single-user ToDo workflow the student can use, backed by a Python/FastAPI backend and SQLite so that data survives restarts.

## What Changes

- Introduce a static, framework-free frontend (HTML, CSS, vanilla JavaScript) with four public pages: Homepage, About, Portfolio, and Contact.
- Introduce a private ToDo page that lists, creates, edits, completes, and deletes ToDo items.
- Introduce a FastAPI backend that owns all business logic and all database access, exposed as a REST/JSON API consumed by the frontend.
- Introduce a SQLite database storing ToDo items (title, description, completion status, creation date), created and migrated automatically on startup so items persist across restarts.
- Seed the site's content (biography, education, skills, interests, projects, contact details) as static frontend content; no admin UI for editing site content in this version.
- Out of scope: user registration, multiple users, roles, authentication, a contact form, and any database access from the browser.

## Capabilities

### New Capabilities

- `student-profile`: The Homepage and About pages — short introduction, site overview, biography, education, skills, and interests.
- `project-portfolio`: The Portfolio page — the list of student projects with title, description, technologies used, and an optional project or source-code link.
- `contact-info`: The Contact page — the student's contact details presented without a form.
- `todo-list`: The private ToDo list behavior — viewing, creating, editing, completing, and deleting items, and their persistence across application restarts.
- `todo-api`: The REST/JSON API contract the backend exposes for ToDo items, including request/response shapes and error behavior.

### Modified Capabilities

None. This is the first change in the project; `openspec/specs/` is empty.

## Impact

- New code: `/backend` (FastAPI application, SQLite access layer, request/response models) and `/frontend` (four public pages, the ToDo page, shared CSS and JavaScript).
- New runtime dependencies: Python with FastAPI, an ASGI server, and a SQLite driver.
- Backend serves the REST API and the static frontend so the site runs from a single process.
- No existing code, APIs, or data are affected; the repository has no implementation yet.
