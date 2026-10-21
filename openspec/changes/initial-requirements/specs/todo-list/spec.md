# Spec Delta

## Purpose

Lets the single owner of this private ToDo page view, create, edit, complete, and delete ToDo items, and keeps those items after the application restarts.

## ADDED Requirements

### Requirement: ToDo item attributes

Each ToDo item SHALL have a title, a description, a completion status, and a creation date.

#### Scenario: Item fields are visible in the list

- **WHEN** the ToDo list is displayed
- **THEN** each item shows its title and completion status
- **AND** the item's description and creation date are available on that item's entry

#### Scenario: Completion status is unambiguous

- **WHEN** an item is displayed
- **THEN** its completion status is distinguishable without opening a detail view

### Requirement: View all ToDo items

The ToDo page SHALL display all stored ToDo items.

#### Scenario: Items exist

- **WHEN** the ToDo page is opened and items are stored
- **THEN** every stored item is listed

#### Scenario: No items exist

- **WHEN** the ToDo page is opened and no items are stored
- **THEN** the page indicates that the list is empty
- **AND** the create action remains available

#### Scenario: Ordering of items

- **WHEN** the list is displayed
- **THEN** items appear in a stable order based on their creation date, newest first

### Requirement: Create a ToDo item

The ToDo page SHALL let the user create an item by providing at least a title, with an optional description.

#### Scenario: Create with valid input

- **WHEN** the user submits a new item with a non-empty title
- **THEN** the item is stored and appears in the list
- **AND** its completion status is not completed
- **AND** its creation date is set at the time it is created

#### Scenario: Title is required

- **WHEN** the user submits a new item with an empty or whitespace-only title
- **THEN** the item is not stored
- **AND** the page shows a message explaining that a title is required

### Requirement: Edit a ToDo item

The ToDo page SHALL let the user change the title and description of an existing item.

#### Scenario: Edit an existing item

- **WHEN** the user edits an item and saves a non-empty title
- **THEN** the updated title and description are stored and shown in the list
- **AND** the item's creation date and completion status are unchanged

#### Scenario: Reject an invalid edit

- **WHEN** the user submits an edit with an empty or whitespace-only title
- **THEN** the change is not stored
- **AND** the page reports that a title is required

#### Scenario: Cancel an edit

- **WHEN** the user abandons an edit before saving
- **THEN** the stored item is unchanged

### Requirement: Mark a ToDo item as completed

The ToDo page SHALL let the user mark an item as completed and reverse that mark.

#### Scenario: Mark as completed

- **WHEN** the user marks an incomplete item as completed
- **THEN** the item is stored as completed
- **AND** the list shows it as completed

#### Scenario: Mark as not completed

- **WHEN** the user marks a completed item as not completed
- **THEN** the item is stored as not completed
- **AND** the list shows it as not completed

### Requirement: Delete a ToDo item

The ToDo page SHALL let the user delete an item.

#### Scenario: Delete an item

- **WHEN** the user deletes a list item and confirms the deletion
- **THEN** the item is removed from storage
- **AND** it no longer appears in the list

#### Scenario: Deletion requires confirmation

- **WHEN** the user starts a deletion but does not confirm it
- **THEN** the item remains stored and visible

### Requirement: Persistence across restarts

ToDo items SHALL remain available after the application is stopped and started again.

#### Scenario: Items survive a restart

- **WHEN** items have been stored and the application is restarted
- **THEN** the ToDo page lists the same items with their titles, descriptions, completion status, and creation dates unchanged

#### Scenario: First start creates storage

- **WHEN** the application starts and its database does not exist yet
- **THEN** the storage is created automatically
- **AND** the ToDo page reports an empty list instead of failing

### Requirement: Private ToDo page

The ToDo page SHALL be reachable directly but SHALL NOT be part of the public site navigation.

#### Scenario: Visitor browses the public site

- **WHEN** a visitor views the Homepage, About, Portfolio, or Contact page
- **THEN** no navigation link to the ToDo page is presented

### Requirement: Failures are reported to the user

The ToDo page SHALL report an unsuccessful create, update, complete, or delete attempt instead of silently leaving the list out of date.

#### Scenario: Storage request fails

- **WHEN** an operation on a ToDo item fails
- **THEN** the page displays a message describing the failure
- **AND** the displayed list still reflects the stored items
