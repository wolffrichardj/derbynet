# DerbyNet - AI Agent Guidance

This document provides guidance for AI agents working with the DerbyNet codebase.

## Project Overview

DerbyNet is a web-based race management system for Pinewood Derby events. It consists of:
- **PHP web application** (`website/`) - Main user interface and business logic
- **Timer interfaces** (`timer/`) - Java and JavaScript applications for hardware timer communication
- **Build system** - Apache Ant-based build configuration

## Documentation Location

**Developer Documentation (Markdown):**
- `README.md` - Project overview and quick start
- `ARCHITECTURE.md` - System architecture and component details
- `DEVELOPMENT.md` - Development setup and build process
- `agents.md` - This file (AI agent guidance)

**User Documentation (PDF):**
- `docs/` directory contains ODF source files that generate PDF documentation
- PDFs are generated during build process (requires LibreOffice)
- Key topics: Installation, Timer Operation, Running a Race, etc.
- See `docs/build.xml` for PDF generation targets

**Code Documentation:**
- PHP: Inline comments and function documentation
- JavaScript: Comments in source files
- Java: Javadoc-style comments

## Key Directories

**Web Application:**
- `website/ajax/` - AJAX action handlers (107 files, pattern: `action.*.inc`)
- `website/inc/` - Shared PHP includes (database, auth, utilities)
- `website/js/` - Client-side JavaScript
- `website/sql/` - Database schema files

**Timer:**
- `timer/src/org/jeffpiazza/derby/` - Java timer source
- `timer/derbynet-timer/src/` - JavaScript timer source

**Configuration:**
- `build.xml` - Apache Ant build configuration
- `website/local/` - Runtime configuration (created during setup)

## Common Tasks

### Finding Code

**Database Operations:**
- Database connection: `website/inc/data.inc`
- Schema management: `website/inc/schema_version.inc`
- SQL queries: Search `website/ajax/` and `website/inc/` for SQL patterns

**Timer Communication:**
- Timer message handler: `website/ajax/action.timer-message.inc`
- Timer state: `website/inc/timer-state.inc`
- Java timer: `timer/src/org/jeffpiazza/derby/TimerMain.java`
- Browser timer: `timer/derbynet-timer/src/js/host_poller.js`

**User Interface:**
- Main pages: `website/*.php` files
- AJAX endpoints: `website/ajax/action.*.inc` files
- JavaScript: `website/js/*.js` files

**Authentication:**
- Authorization: `website/inc/authorize.inc`
- Permissions: `website/inc/permissions.inc`
- Session management: Uses PHP sessions

### Understanding Data Flow

1. **Timer → Server:**
   - Timer sends HTTP POST to `action.php?action=timer-message`
   - Handler: `website/ajax/action.timer-message.inc`
   - Updates database and race state

2. **Web UI → Server:**
   - AJAX requests to `action.php`
   - Routed to `website/ajax/action.*.inc` handlers
   - Returns JSON or XML responses

3. **Database Access:**
   - PDO-based abstraction in `website/inc/data.inc`
   - Supports SQLite, MySQL, ODBC
   - Schema versioning system

### Code Patterns

**PHP:**
- Uses PDO for database access
- Prepared statements for queries
- `htmlspecialchars()` for output escaping
- Session-based authentication
- Permission checks via `have_permission()`

**JavaScript:**
- jQuery-based (legacy codebase)
- Event-driven architecture for timer
- AJAX for server communication

**Java:**
- Timer uses message queue pattern
- HTTP communication via `ClientSession`
- Device abstraction via `TimerDevice` interface

## Important Files

**Configuration:**
- `website/local/config-database.inc` - Database connection (runtime)
- `extras/derbynet.conf` - Timer configuration
- `build.xml` - Build configuration

**Core Logic:**
- `website/inc/data.inc` - Database access functions
- `website/inc/racing-state.inc` - Race state management
- `website/inc/timer-state.inc` - Timer state tracking
- `website/action.php` - AJAX router

**Entry Points:**
- `website/index.php` - Main page
- `website/setup.php` - Initial configuration
- `website/coordinator.php` - Race coordination
- `website/timer.php` - Browser timer interface

## Build System

**Key Commands:**
- `ant generated` - Generate version files
- `ant timer-jar` - Build Java timer
- `ant timer-in-browser` - Build browser timer
- `ant clean` - Remove generated files

**Generated Files:**
- Version info in `website/inc/generated-*.inc`
- Timer profiles in `website/js/timer/profiles.js`
- Banner in `website/inc/banner.inc`

## Database Schema

**Key Tables:**
- `Roster` - Racer information
- `RaceChart` - Heat scheduling and results
- `Classes` - Racing divisions
- `Rounds` - Racing rounds
- `RaceInfo` - System configuration
- `Awards` - Award definitions

**Schema Files:**
- `website/sql/sqlite/` - SQLite schema
- `website/sql/access/` - Access/ODBC schema
- Schema version tracked in `RaceInfo` table

## Testing

**Test Utilities:**
- `website/fake-timer.php` - Simulate timer without hardware
- `website/fakeroster.php` - Generate test racer data
- `timer/src/org/jeffpiazza/derby/devices/SimulatedDevice.java` - Simulated timer device

## Common Issues

**Database:**
- Connection issues: Check `local/config-database.inc`
- Schema version: See `website/inc/schema_version.inc`
- Permissions: Ensure database file is writable (SQLite)

**Timer:**
- Connection: Verify network connectivity
- Configuration: Check `derbynet.conf` file
- Messages: Review `action.timer-message.inc` for protocol

**Build:**
- Missing generated files: Run `ant generated`
- Timer build fails: Check Java JDK installation
- Docs not generated: LibreOffice not required, silently skipped

## Quick Reference

**Find AJAX handler:** `website/ajax/action.{action-name}.inc`
**Find page:** `website/{page-name}.php`
**Find JavaScript:** `website/js/{feature}.js`
**Find database code:** `website/inc/data.inc` and `website/ajax/`
**Find timer code:** `timer/src/` (Java) or `timer/derbynet-timer/src/` (JavaScript)

**For detailed information, refer to:**
- `ARCHITECTURE.md` - System design and components
- `DEVELOPMENT.md` - Development workflow and build process
- `docs/` directory - User documentation (PDF files)
