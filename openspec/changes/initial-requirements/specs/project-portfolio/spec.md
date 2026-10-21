# Spec Delta

## Purpose

Shows the student's projects on a Portfolio page so a visitor can see what the student built, which technologies were used, and where to find each project.

## ADDED Requirements

### Requirement: Project listing

The Portfolio page SHALL list the student's projects, each with a title, a description, and the technologies used.

#### Scenario: Visitor opens the Portfolio page

- **WHEN** a visitor navigates to the Portfolio page
- **THEN** every project is displayed with its title, description, and technologies used
- **AND** projects are visually separated from one another

#### Scenario: Portfolio contains no projects

- **WHEN** the Portfolio page is displayed with an empty project list
- **THEN** the page shows a message stating that no projects are available yet
- **AND** the page does not display an error

### Requirement: Optional project link

A project MAY expose a link to the running project or to its source code, and the link SHALL be presented as optional per project.

#### Scenario: Project has a link

- **WHEN** a project includes a project or source-code URL
- **THEN** the project entry shows an actionable link labelled for that destination

#### Scenario: Project has no link

- **WHEN** a project does not include a URL
- **THEN** the project entry still displays its title, description, and technologies
- **AND** no empty or broken link is rendered

### Requirement: Technologies are distinguishable

Each project SHALL present its technologies used as separate identifiable values rather than one unstructured sentence.

#### Scenario: Visitor reads a project's technologies

- **WHEN** a project lists several technologies
- **THEN** each technology is displayed as its own item

### Requirement: Portfolio is reachable from the public site

The Portfolio page SHALL be reachable from the navigation of every public page.

#### Scenario: Visitor opens the Portfolio from another page

- **WHEN** a visitor selects "Portfolio" in the navigation
- **THEN** the Portfolio page loads and its content is visible without scrolling past the navigation
