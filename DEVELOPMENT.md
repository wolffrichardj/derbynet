# DerbyNet Development Guide

## Prerequisites

**Required:**
- [Apache Ant](https://ant.apache.org/) 1.9+
- Git
- Java JDK 8+ (for building timer)
- PHP 7.4+ (for running web application locally)

**Optional:**
- LibreOffice/OpenOffice (for generating PDF docs from ODF sources)
- Docker (for containerized development)

**Note:** For user installation instructions, see `docs/` directory (PDF files).

## Development Setup

### 1. Clone Repository

```bash
git clone https://github.com/jeffpiazza/derbynet.git
cd derbynet
```

### 2. Build Generated Files

Generate version information and other build artifacts:

```bash
ant generated
```

This creates:
- `website/inc/generated-version.inc` - Git revision count
- `website/inc/generated-build-date.inc` - Build date
- `website/inc/generated-commit-hash.inc` - Git commit hash
- `website/inc/banner.inc` - Version banner

**Note:** PDF documentation generation requires LibreOffice/OpenOffice. If not available, this step is skipped.

### 3. Build Timer Components

**Browser Timer:**
```bash
ant timer-in-browser
```
Builds JavaScript timer interface and copies to `website/js/timer/`

**Java Timer:**
```bash
ant timer-jar
```
Compiles Java timer application to `timer/dist/lib/derby-timer.jar`

### 4. Set Up Web Server

#### Option A: Docker (Recommended)

Use Docker to provide web server and PHP without local installation:

```bash
# Build generated files first
ant generated timer-in-browser

# Run Docker container with local sources
docker run --detach -p 80:80 -p 443:443 \
  --volume /path/to/data:/var/lib/derbynet \
  --mount type=bind,src=$(pwd)/website,target=/var/www/html,readonly \
  jeffpiazza/derbynet_server
```

**Note:** Use `readonly` mount for development to prevent container from modifying sources.

#### Option B: Local PHP Server

1. Install PHP and web server (Apache/nginx)
2. Configure web server to serve `website/` directory
3. Set environment variables:
   ```bash
   export DERBYNET_CONFIG_DIR=/path/to/config
   export DERBYNET_DATA_DIR=/path/to/data
   ```
4. Create `local/` directory in `website/` for configuration

#### Option C: PHP Built-in Server

For quick testing:

```bash
cd website
php -S localhost:8000
```

Access at `http://localhost:8000`

## Project Structure

```
derbynet/
├── website/          # PHP web application
│   ├── ajax/         # AJAX action handlers
│   ├── inc/          # PHP includes (shared code)
│   ├── js/           # JavaScript files
│   ├── css/          # Stylesheets
│   └── sql/          # Database schema files
├── timer/            # Timer interface
│   ├── src/          # Java source code
│   └── derbynet-timer/src/  # JavaScript timer source
├── docs/             # User documentation (ODF sources)
├── installer/        # Installation scripts and packages
├── extras/           # Additional scripts and resources
└── build.xml         # Apache Ant build configuration
```

## Build Targets

**Common Targets:**

- `ant generated` - Generate version files and banner
- `ant timer-jar` - Build Java timer JAR
- `ant timer-in-browser` - Build browser timer interface
- `ant timer-electron` - Build Electron-based timer app
- `ant docs-dist` - Generate PDF documentation (requires LibreOffice)
- `ant clean` - Remove generated files
- `ant dist` - Build all distribution packages

**Platform-Specific:**

- `ant dist.debian` - Build Debian package
- `ant dist.mac` - Build macOS installer (macOS only)
- `ant dist.oneclick` - Build Windows distribution (Windows only)

## Development Workflow

### Making Changes

1. **Web Application Changes:**
   - Edit PHP files in `website/`
   - Edit JavaScript in `website/js/`
   - Edit CSS in `website/css/`
   - Changes take effect immediately (no rebuild needed)

2. **Timer Changes:**
   - **Java Timer:** Edit Java source in `timer/src/`, run `ant timer-jar`
   - **Browser Timer:** Edit JavaScript in `timer/derbynet-timer/src/`, run `ant timer-in-browser`

3. **Database Schema Changes:**
   - Update SQL files in `website/sql/sqlite/` and `website/sql/access/`
   - Increment schema version in `website/inc/schema_version.inc`
   - Add migration logic in `website/inc/schema-migrations.inc`

### Testing

**Manual Testing:**
- Use `fake-timer.php` to simulate timer without hardware
- Use `fakeroster.php` to generate test racer data
- Test with SQLite database for simplicity

**Timer Testing:**
- Use browser timer (`timer.php`) for quick testing
- Use Java timer with simulated device for hardware testing
- See `timer/src/org/jeffpiazza/derby/devices/SimulatedDevice.java`

### Code Style

**PHP:**
- Follow PSR-12 coding standards
- Use PDO prepared statements for database queries
- Escape output with `htmlspecialchars()`

**JavaScript:**
- Use ES6+ features where supported
- Follow existing code patterns
- Use `'use strict';` in new files

**Java:**
- Follow Java naming conventions
- Document public APIs
- Use existing logging infrastructure (`LogWriter`)

## Key Development Areas

### Adding New Features

1. **Web UI Feature:**
   - Add PHP page in `website/`
   - Add AJAX handler in `website/ajax/action.*.inc`
   - Add JavaScript in `website/js/`
   - Update navigation/permissions as needed

2. **Timer Feature:**
   - Add device support in `timer/src/org/jeffpiazza/derby/devices/`
   - Add profile in `timer/src/org/jeffpiazza/derby/profiles/`
   - Update browser timer JavaScript if needed

3. **Database Feature:**
   - Add schema changes in `website/sql/`
   - Update schema version
   - Add migration logic

### Debugging

**PHP Debugging:**
- Enable error reporting in `local/config-database.inc`:
  ```php
  error_reporting(E_ALL);
  ini_set('display_errors', 1);
  ```
- Check web server error logs
- Use `about.php` page for diagnostic information

**Timer Debugging:**
- Enable verbose logging in timer:
  ```bash
  java -jar derby-timer.jar -trace http://server
  ```
- Check browser console for browser timer
- Review timer log files in data directory

**Database Debugging:**
- Use SQLite browser for SQLite databases
- Check `RaceInfo` table for system state
- Review schema version in `about.php`

## Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature-name`
3. Make changes and test thoroughly
4. Commit with descriptive messages
5. Push to fork and create pull request

**Before Submitting:**
- Run `ant clean generated` to ensure clean build
- Test on multiple browsers/platforms if UI changes
- Update documentation if needed
- Follow existing code patterns

## Resources

- **User Documentation:** `docs/` directory (PDF files)
- **API Documentation:** See `website/ajax/` for AJAX endpoints
- **Timer Protocol:** See `website/ajax/action.timer-message.inc` for message format
- **Database Schema:** See `website/sql/` for table definitions
