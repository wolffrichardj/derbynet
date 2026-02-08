# DerbyNet

![icon](https://raw.githubusercontent.com/jeffpiazza/derbynet/master/website/img/derbynet-300.png)

**DerbyNet** is a free, open-source, web-based race management system for Pinewood Derby-style racing events. It provides comprehensive tools for managing racers, coordinating races, capturing timer data, displaying results, and presenting awards.

## Overview

DerbyNet is a multi-screen race management solution that enables:
- **Racer Management**: Check-in, roster import/export, photo capture
- **Race Coordination**: Heat scheduling, lane assignments, racing groups
- **Timer Integration**: Support for multiple hardware timer devices via serial/USB
- **Results Display**: Real-time race results, standings, and history
- **Awards Management**: Award creation, online balloting, and presentation displays
- **Kiosk Mode**: Dedicated displays for check-in, "now racing", standings, and more

## Key Features

- **Web-Based Interface**: Accessible from any device with a web browser
- **Multi-Timer Support**: Java-based timer interface (derby-timer.jar) and browser-based timer
- **Database Flexibility**: Supports SQLite (default), MySQL, and ODBC (for GPRM compatibility)
- **Multi-Screen Support**: Coordinate displays for different purposes (racing, standings, check-in)
- **Photo Integration**: Capture and display racer photos
- **Export/Import**: JSON export, roster import, snapshot/restore functionality

## Documentation

**For Users:**
- Installation and user guides: See `docs/` directory (PDF files generated from ODF sources)
- **[DREAMHOST.md](DREAMHOST.md)** - Dreamhost-specific setup and configuration
- Visit [https://derbynet.org](https://derbynet.org) for installation packages

**For Developers:**
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - System architecture and component overview
- **[DEVELOPMENT.md](DEVELOPMENT.md)** - Development setup and build instructions
- **[agents.md](agents.md)** - Guidance for AI agents working with this codebase

## Quick Start

### For Users

Visit [https://derbynet.org](https://derbynet.org) for installation packages and user guides.

### For Developers

See [DEVELOPMENT.md](DEVELOPMENT.md) for detailed development setup instructions.

Quick local development setup:

1. Install [Apache Ant](https://ant.apache.org/)
2. Run `ant generated` to build generated files
3. Build timer components: `ant timer-in-browser` and/or `ant timer-jar`
4. Use Docker for web server/PHP: See [DEVELOPMENT.md](DEVELOPMENT.md) for details

## Deployment

This repository includes a GitHub Actions workflow for deploying to Dreamhost via SFTP.

For detailed instructions on configuring deployment, setting up SSH keys, and managing secrets, please see **[DREAMHOST.md](DREAMHOST.md)**.

## License

MIT License - see [MIT-LICENSE.txt](MIT-LICENSE.txt)
