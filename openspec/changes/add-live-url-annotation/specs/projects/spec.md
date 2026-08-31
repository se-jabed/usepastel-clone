## Purpose

Lets an authenticated owner create and manage projects that point at a target URL, and gives each project a shareable link that guests use to view and annotate it.

## ADDED Requirements

### Requirement: Owner can create a project
An authenticated owner SHALL be able to create a project by submitting a target URL. The system SHALL reject creation if the URL is not a well-formed absolute `http://` or `https://` URL.

#### Scenario: Successful project creation
- **WHEN** an authenticated owner submits a valid absolute URL
- **THEN** the system creates a project owned by that owner and associated with the submitted URL

#### Scenario: Rejects invalid target URL
- **WHEN** an authenticated owner submits a value that is not a well-formed absolute http(s) URL
- **THEN** the system rejects creation and returns a validation error without creating a project

#### Scenario: Unauthenticated users cannot create projects
- **WHEN** an unauthenticated request attempts to create a project
- **THEN** the system rejects the request and no project is created

### Requirement: Project has a unique shareable guest link
Every project SHALL have a unique, unguessable share link that, when opened, resolves to the proxied rendering of that project's target URL.

#### Scenario: Guest link resolves to the correct project
- **WHEN** any user opens a project's share link
- **THEN** the system resolves it to that project and serves the proxied rendering of its target URL

#### Scenario: Share links are unique per project
- **WHEN** two different projects are created
- **THEN** their share links are distinct and neither resolves to the other's project

### Requirement: Owner can list their own projects
An authenticated owner SHALL be able to retrieve a list of projects they created. The list SHALL NOT include projects owned by other accounts.

#### Scenario: Dashboard shows only the owner's projects
- **WHEN** an authenticated owner requests their project list
- **THEN** the system returns only projects created by that owner
