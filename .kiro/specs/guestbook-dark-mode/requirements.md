# Requirements Document

## Introduction

This feature adds a dark mode theme to the guestbook (留言板) section of the GCLI2API control panel. The dark mode will provide a visually comfortable experience for users in low-light environments while maintaining all existing functionality.

## Glossary

- **Guestbook**: The message board section where users can leave comments and messages
- **Dark Mode**: A color scheme that uses light-colored text and UI elements on dark backgrounds
- **Control Panel**: The web-based management interface for GCLI2API
- **Theme Toggle**: A UI control that allows users to switch between light and dark modes

## Requirements

### Requirement 1

**User Story:** As a user, I want to view the guestbook in dark mode, so that I can comfortably read messages in low-light environments.

#### Acceptance Criteria

1. WHEN the guestbook tab is displayed THEN the system SHALL apply dark theme colors to all UI elements
2. WHEN displaying message cards THEN the system SHALL use dark backgrounds with light text for readability
3. WHEN displaying the chat header THEN the system SHALL maintain the gradient design with adjusted colors for dark mode
4. WHEN displaying the input area THEN the system SHALL use dark backgrounds with appropriate contrast for input fields
5. WHEN displaying emoji picker THEN the system SHALL apply dark theme styling to the picker panel

### Requirement 2

**User Story:** As a user, I want the dark mode to match the reference design, so that the interface looks consistent and professional.

#### Acceptance Criteria

1. WHEN dark mode is active THEN the main background SHALL use color #2c3440 or similar dark gray
2. WHEN dark mode is active THEN the message cards SHALL use color #3a4350 or similar for backgrounds
3. WHEN dark mode is active THEN text SHALL use rgba(255, 255, 255, 0.9) or similar light colors
4. WHEN dark mode is active THEN borders SHALL use rgba(255, 255, 255, 0.12) or similar subtle colors
5. WHEN dark mode is active THEN the header gradient SHALL remain visible with adjusted opacity

### Requirement 3

**User Story:** As a user, I want all interactive elements to work properly in dark mode, so that I can use all features without issues.

#### Acceptance Criteria

1. WHEN hovering over buttons in dark mode THEN the system SHALL display appropriate hover effects
2. WHEN focusing on input fields in dark mode THEN the system SHALL show clear focus indicators
3. WHEN displaying the emoji picker in dark mode THEN the system SHALL maintain proper contrast and visibility
4. WHEN displaying message timestamps in dark mode THEN the system SHALL use readable text colors
5. WHEN displaying the character counter in dark mode THEN the system SHALL use appropriate text colors

### Requirement 4

**User Story:** As a user, I want the dark mode to be applied automatically, so that I don't need to manually toggle it.

#### Acceptance Criteria

1. WHEN the page loads THEN the system SHALL apply dark mode styling to the guestbook section
2. WHEN switching to the guestbook tab THEN the system SHALL ensure dark mode is active
3. WHEN the guestbook renders THEN the system SHALL apply all dark mode styles consistently
4. WHEN new messages are added THEN the system SHALL render them with dark mode styling
5. WHEN the page refreshes THEN the system SHALL maintain dark mode styling
