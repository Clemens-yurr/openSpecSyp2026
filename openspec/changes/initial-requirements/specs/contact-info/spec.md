# Spec Delta

## Purpose

Gives visitors a Contact page with the student's contact details and destinations they can act on, without a contact form.

## ADDED Requirements

### Requirement: Contact details

The Contact page SHALL display the student's contact information, including at least an email address.

#### Scenario: Visitor opens the Contact page

- **WHEN** a visitor navigates to the Contact page
- **THEN** the page displays the student's email address as visible text
- **AND** additional contact channels, when present, are each labelled with their type

### Requirement: Actionable contact channels

Each contact channel SHALL be presented in a form the visitor can act on immediately.

#### Scenario: Visitor selects the email address

- **WHEN** the email address is displayed
- **THEN** it is a link that opens the visitor's email client addressed to the student

#### Scenario: Visitor selects an external profile link

- **WHEN** a contact channel points to an external profile such as a code hosting or professional network page
- **THEN** it opens in the browser without navigating the website away

### Requirement: No contact form

The Contact page SHALL NOT present a submission form.

#### Scenario: Visitor looks for a form

- **WHEN** the Contact page is displayed
- **THEN** no input fields, message textarea, or submit control are shown

### Requirement: Contact page is reachable from the public site

The Contact page SHALL be reachable from the navigation of every public page.

#### Scenario: Visitor opens the Contact page from another page

- **WHEN** a visitor selects "Contact" in the navigation
- **THEN** the Contact page loads with the contact details visible
