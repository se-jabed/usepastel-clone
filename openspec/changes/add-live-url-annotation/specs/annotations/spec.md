## Purpose

Lets anyone viewing a proxied project page place pins, write threaded comments, tag, and resolve feedback, with changes synced live to everyone else viewing the same project.

## ADDED Requirements

### Requirement: Viewers can place a pin with a comment
Any viewer of a proxied project page, including guests with no account, SHALL be able to click a location on the page to drop a pin and attach a comment to it. The pin's anchor SHALL be stored as a CSS selector plus a relative offset within that element, not as raw pixel or viewport coordinates.

#### Scenario: Guest places a pin and comment
- **WHEN** a guest with no account clicks a location on a proxied project page and submits comment text
- **THEN** the system creates a pin anchored to that location's element and offset, with the submitted comment attached

#### Scenario: Pin anchor survives viewport changes
- **WHEN** a pin created at one viewport width is viewed at a different viewport width or scroll position
- **THEN** the pin renders anchored to the same element it was originally placed on

### Requirement: Comments support threaded replies
A viewer SHALL be able to reply to an existing comment, creating a threaded conversation under the original pin.

#### Scenario: Reply to an existing comment thread
- **WHEN** a viewer submits a reply to a comment on an existing pin
- **THEN** the reply is stored and displayed as part of that pin's comment thread, in order

### Requirement: Comments can be tagged
An authenticated owner SHALL be able to attach one or more free-form text tags to a comment thread.

#### Scenario: Owner adds a tag to a comment thread
- **WHEN** an authenticated owner adds a tag to a comment thread
- **THEN** the tag is stored and displayed alongside that thread

### Requirement: Comment threads can be marked resolved or unresolved
A viewer SHALL be able to mark a comment thread as resolved, and later mark it unresolved again. Resolved threads SHALL be distinguishable from unresolved ones.

#### Scenario: Mark a thread resolved
- **WHEN** a viewer marks an unresolved comment thread as resolved
- **THEN** the thread's status changes to resolved and is reflected to all viewers

#### Scenario: Reopen a resolved thread
- **WHEN** a viewer marks a resolved comment thread as unresolved
- **THEN** the thread's status changes back to unresolved

### Requirement: Guests can comment without an account
The system SHALL allow a viewer with no account to place pins and write comments, identified by a display name they provide, without requiring sign-up or login.

#### Scenario: Guest submits a comment with only a display name
- **WHEN** a guest provides a display name and submits a comment without creating an account
- **THEN** the comment is created and attributed to that display name

### Requirement: Pin and comment changes sync live to other viewers
When one viewer creates, replies to, tags, or changes the resolution status of a pin/comment, every other viewer currently viewing the same project SHALL see that change without reloading the page.

#### Scenario: Second viewer sees a new pin appear live
- **WHEN** one viewer creates a new pin while a second viewer is already viewing the same project
- **THEN** the second viewer sees the new pin appear without refreshing the page
