# Spec Delta

## Purpose

Defines the REST/JSON interface between the browser frontend and the backend for ToDo items, so the frontend can manage items without touching the database.

## ADDED Requirements

### Requirement: ToDo collection and item endpoints

The backend SHALL expose REST endpoints to list ToDo items, create an item, update an existing item, and delete an item.

#### Scenario: List items

- **WHEN** the frontend sends a GET request for the ToDo collection
- **THEN** the backend responds with a success status and a JSON array of items, empty when no items are stored

#### Scenario: Create an item

- **WHEN** the frontend sends a POST request for the ToDo collection with a valid item payload
- **THEN** the backend stores the item and responds with a success status and the created item, including its identifier and creation date

#### Scenario: Update an item

- **WHEN** the frontend sends a request to update an existing item with a valid payload
- **THEN** the backend stores the change and responds with the updated item

#### Scenario: Delete an item

- **WHEN** the frontend sends a delete request for an existing item
- **THEN** the backend removes the item and responds with a success status

### Requirement: Item representation in JSON

Each ToDo item in a response SHALL contain an identifier, a title, a description, a completion status, and a creation date.

#### Scenario: Inspect a returned item

- **WHEN** an item is returned by any ToDo endpoint
- **THEN** the payload contains all five attributes
- **AND** the creation date is a machine-readable timestamp
- **AND** the completion status is a boolean

### Requirement: Validation errors are reported

The API SHALL reject invalid ToDo payloads with a client error and a JSON body describing the problem.

#### Scenario: Missing title on create

- **WHEN** a create request has no title or a whitespace-only title
- **THEN** the backend responds with a client error status
- **AND** the response body identifies the offending field

#### Scenario: Invalid completion status

- **WHEN** an update request supplies a completion status that is not a boolean
- **THEN** the backend responds with a client error status
- **AND** the item is not changed

### Requirement: Unknown item identifier

Requests targeting an item identifier that does not exist SHALL be answered with a not-found error rather than a success.

#### Scenario: Update an unknown item

- **WHEN** the frontend sends an update for an identifier that is not stored
- **THEN** the backend responds with a not-found status

#### Scenario: Delete an unknown item

- **WHEN** the frontend sends a delete for an identifier that is not stored
- **THEN** the backend responds with a not-found status
- **AND** the rest of the stored items are unaffected

### Requirement: Frontend and database separation

The frontend SHALL reach ToDo data only through these HTTP endpoints, and the backend SHALL perform all business logic and database access.

#### Scenario: Frontend stores an item

- **WHEN** the user creates, edits, completes, or deletes an item in the browser
- **THEN** the change is sent to the backend over HTTP
- **AND** the browser does not open or modify the database file

#### Scenario: Interactive API documentation

- **WHEN** a developer opens the backend's documentation route in a browser
- **THEN** the available ToDo endpoints are described with their request and response shapes
