# DerbyNet Architecture

## System Overview

DerbyNet is a client-server web application built with PHP, JavaScript, and Java. The system consists of three main components:

1. **Web Application** (`website/`) - PHP-based web interface
2. **Timer Interface** (`timer/`) - Java application and JavaScript browser interface  
3. **Build System** (`build.xml`) - Apache Ant build configuration

**Note:** For installation instructions, see the `docs/` directory (PDF files) or visit [https://derbynet.org](https://derbynet.org).

## Component Architecture

### Web Application (`website/`)

The web application is a PHP-based system that provides the primary user interface and business logic.

**Key Directories:**
- `ajax/` - AJAX endpoint handlers (107 action handlers)
- `inc/` - Shared PHP includes (database, authorization, utilities)
- `js/` - Client-side JavaScript (67 files)
- `css/` - Stylesheets and assets
- `sql/` - Database schema files (SQLite and Access variants)
- `kiosks/` - Kiosk configuration files for different display modes

**Core Functionality:**
- User authentication and role-based permissions
- Database abstraction (SQLite, MySQL, ODBC)
- Race state management
- Timer communication via HTTP polling
- Results processing and display
- Award management and balloting

**Key Files:**
- `index.php` - Main entry point
- `action.php` - AJAX action router
- `coordinator.php` - Race coordination interface
- `setup.php` - Initial database configuration
- `timer.php` - Browser-based timer interface

### Timer Interface

DerbyNet supports two timer interface implementations:

#### Java Timer (`timer/src/`)

A Java application (`derby-timer.jar`) that communicates with hardware timers via serial/USB ports and relays data to the web server via HTTP.

**Architecture:**
- `TimerMain.java` - Entry point and main loop
- `ClientSession.java` - HTTP communication with web server
- `devices/` - Timer device drivers (MicroWizard, RacingStateMachine, etc.)
- `profiles/` - Timer profile configurations for different hardware
- `HttpTask.java` - Message queue and HTTP request handling

**Communication Protocol:**
- Timer sends messages: `HELLO`, `IDENTIFIED`, `STARTED`, `FINISHED`, `HEARTBEAT`, `MALFUNCTION`
- Server responds with: `<HEAT-READY/>`, `<ABORT/>`
- Messages sent via POST to `action.php?action=timer-message`

#### Browser Timer (`timer/derbynet-timer/src/`)

A JavaScript-based timer interface that runs in a web browser, useful for testing or when hardware timer access isn't available.

**Key Components:**
- `main.js` - Main application logic
- `host_poller.js` - HTTP communication with server
- `state_machine.js` - Race state management
- `port_wrapper.js` - Serial port abstraction (when available)

### Database Layer

**Supported Databases:**
- **SQLite** (default) - File-based, no server required
- **MySQL** - For multi-user or high-performance scenarios
- **ODBC** - For compatibility with Grand Prix Race Manager (GPRM)

**Schema Management:**
- Schema versioning system tracks database migrations
- SQL files in `website/sql/sqlite/` and `website/sql/access/`
- Automatic schema upgrades on access

**Key Tables:**
- `Roster` - Racer information
- `RaceChart` - Heat scheduling and results
- `Classes` - Racing classes/divisions
- `Rounds` - Racing rounds
- `RaceInfo` - System configuration and state
- `Awards` - Award definitions
- `BallotAwards` - Online balloting data

### Communication Flow

```
Hardware Timer → Timer Interface (Java/JS) → HTTP POST → Web Server (PHP) → Database
                                                              ↓
                                                         Web UI (Browser)
```

1. Timer interface connects to hardware via serial/USB
2. Timer sends `HELLO` message to web server
3. Server responds with current race state
4. Timer sends race events (`STARTED`, `FINISHED`) as they occur
5. Server processes results and updates database
6. Web UI polls for updates via AJAX

## Build System

**Apache Ant** (`build.xml`) orchestrates the build process:

**Key Targets:**
- `generated` - Generate version info, banner, timer profiles
- `timer-jar` - Build Java timer application
- `timer-in-browser` - Build browser timer interface
- `docs-dist` - Generate PDF documentation from ODF sources
- `dist` - Create distribution packages (Mac, Debian, Windows)

**Generated Files:**
- `website/inc/generated-version.inc` - Git revision count
- `website/inc/generated-build-date.inc` - Build timestamp
- `website/inc/generated-commit-hash.inc` - Git commit hash
- `website/inc/banner.inc` - Version banner
- `website/js/timer/profiles.js` - Timer profile definitions

## Security Model

**Role-Based Access Control:**
- Roles defined in database (`Roles` table)
- Permissions checked via `have_permission()` function
- Session-based authentication
- Default roles: Administrator, Race Coordinator, Timer, Photo, etc.

**Data Protection:**
- SQL injection prevention via PDO prepared statements
- XSS prevention via `htmlspecialchars()`
- Session management for authentication state

## Deployment Options

1. **Docker** (`installer/docker/`) - Containerized deployment with nginx/PHP
2. **Debian Package** (`installer/debian/`) - Native Linux package
3. **Mac Installer** (`installer/mac/`) - macOS package installer
4. **Windows** (`installer/win/`) - Uniform Server-based distribution

## Configuration

**Database Configuration:**
- Stored in `local/config-database.inc` (or `DERBYNET_CONFIG_DIR`)
- Contains PDO connection string
- Created via `setup.php` interface

**Environment Variables:**
- `DERBYNET_CONFIG_DIR` - Configuration directory path
- `DERBYNET_DATA_DIR` - Data directory (databases, photos, slides)

**Timer Configuration:**
- `extras/derbynet.conf` - Timer connection settings
- Timer profiles in `timer/src/org/jeffpiazza/derby/profiles/`
