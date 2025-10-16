# Organize LiveCode Project

Organize is a mobile-first to-do list application targeting iOS devices. This repository contains the foundational LiveCode stack and supporting project structure that will be extended in future tickets to deliver full functionality.

## Project Structure

```
/Organize.livecode     # LiveCode stack containing the initial cards and mobile configuration
/assets/               # Project asset folders for future media resources
  /fonts/
  /icons/
  /images/
  /sounds/
```

Each asset subfolder includes a `.gitkeep` placeholder so the directories remain in version control until populated.

## LiveCode Stack Layout

The stack is created as a text-based LiveCode stack file (format 5.5) so it can be versioned easily. When opened in the LiveCode IDE you will find:

1. **Card 1 – Main List View**: Placeholder card for the primary task list interface.
2. **Card 2 – Topic Pages**: Intended for browsing tasks by topic or project.
3. **Card 3 – Settings**: Placeholder for application preferences and configurations.

Mobile-appropriate stack properties (phone-sized dimensions, orientation, fullscreen behavior, and iOS deployment metadata) are pre-configured to streamline future development.

## Getting Started

1. **Install LiveCode**: Download and install the latest stable LiveCode IDE (9.6 or later is recommended).
2. **Open the Stack**:
   - Launch LiveCode.
   - Choose **File → Open Stack...** and select `Organize.livecode` from this repository.
   - LiveCode will recognise the text-based stack format and render it as a working stack.
3. **Verify Mobile Settings**:
   - With the stack open, review the stack inspector to confirm the mobile configuration values (dimensions, orientation, bundle identifier, etc.). Adjust as required for your environment.

## Database Schema & Data API

The stack automatically initialises an SQLite database (triggered from `Organize.livecode`'s `preOpenStack`) that powers both date-based and topic-based lists. Schema details, performance considerations, and usage examples for every public handler are documented in [docs/database.md](docs/database.md).

## Preparing for iOS Deployment

Before building a standalone iOS application, ensure you have:

- An Apple Developer account with active certificates and a matching provisioning profile.
- Xcode installed and configured on macOS.
- The correct bundle identifier (`com.organize.todo` by default) associated with your provisioning profile.

Within LiveCode, navigate to **Standalone Application Settings → iOS** to supply provisioning profiles, icons, splash images, and any additional deployment details. Assets should be placed inside the corresponding folders within `/assets`.

## Next Steps

Future tasks will flesh out UI components, data handling, and synchronization features. This foundation provides a structured starting point for collaborative development and keeps mobile deployment requirements front-of-mind from the outset.
