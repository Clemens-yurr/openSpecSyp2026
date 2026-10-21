# Spec Delta

## Purpose

Presents the student behind the website: a homepage introduction with a site overview and an about page covering biography, education, skills, and interests.

## ADDED Requirements

### Requirement: Homepage introduction

The Homepage SHALL present a short introduction to the student and an overview of what the website contains.

#### Scenario: Visitor opens the homepage

- **WHEN** a visitor navigates to the site root
- **THEN** the page displays the student's name and a short introduction
- **AND** it lists the site's sections (About, Portfolio, Contact) as navigable links

#### Scenario: Homepage introduction is brief

- **WHEN** a visitor reads the homepage
- **THEN** the introduction is a summary only, with the full biography available on the About page

### Requirement: About page biography

The About page SHALL present the student's biography, education, skills, and interests.

#### Scenario: Visitor opens the About page

- **WHEN** a visitor navigates to the About page
- **THEN** the page displays a biography section, an education section, a skills section, and an interests section under separate visible headings

#### Scenario: About page is reachable from every public page

- **WHEN** a visitor is on any public page
- **THEN** a link to the About page is available in the site navigation

### Requirement: Site navigation consistency

Every public page SHALL share the same navigation allowing a visitor to reach the Homepage, About, Portfolio, and Contact pages.

#### Scenario: Visitor moves between public pages

- **WHEN** a visitor selects a section from the navigation on any public page
- **THEN** the browser loads that section's page
- **AND** the navigation remains visible on the loaded page

#### Scenario: Current page is indicated

- **WHEN** a public page is displayed
- **THEN** the navigation marks the link for that page as the current section

### Requirement: Public content requires no login

All public pages SHALL be readable without authentication.

#### Scenario: Visitor reads the public site anonymously

- **WHEN** a visitor without any prior session opens any public page
- **THEN** the page content is fully displayed
- **AND** the visitor is not prompted for credentials

### Requirement: Content is usable on small screens

Public pages SHALL remain readable on narrow viewports typical of mobile devices.

#### Scenario: Visitor opens the site on a phone-sized viewport

- **WHEN** the viewport width is 400 pixels
- **THEN** text and navigation are readable without horizontal scrolling
