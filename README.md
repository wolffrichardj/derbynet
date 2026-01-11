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

## Deployment to Dreamhost

### Prerequisites

- A Dreamhost hosting account with SSH access
- GitHub repository with Actions enabled
- Local machine with SSH tools installed

### 1. Generate & Add SSH Key

**Check if you already have an SSH key:**
```bash
ls -la ~/.ssh/id_ed25519*
```

If you already have a key and want to keep it, either:
- Use that existing key, or
- Generate a new key with a different filename (e.g., `~/.ssh/id_ed25519_dreamhost`)

**Generate a new SSH key:**
```bash
# Default location (will prompt to overwrite if exists)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Or specify a custom filename to avoid overwriting
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_dreamhost -C "your_email@example.com"
```

**Note:** If using a custom filename, you'll need to specify it with `-i` when using SSH:
```bash
ssh -i ~/.ssh/id_ed25519_dreamhost user@server
```

### 2. Add Public Key to Dreamhost Server

On your Dreamhost server, add the public key to authorized keys:

```bash
# SSH into your Dreamhost server
ssh username@your-server.dreamhost.com

# Create .ssh directory if it doesn't exist
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add your public key (paste the contents of id_ed25519.pub)
echo "ssh-ed25519 AAAA... your_email@example.com" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 3. Test SSH Connection

After adding the public key to the server, test the connection:

```bash
# Using default key
ssh username@your-server.dreamhost.com

# Or using custom key
ssh -i ~/.ssh/id_ed25519_dreamhost username@your-server.dreamhost.com
```

If successful, you should be logged into the server without a password prompt.

### 4. Add GitHub Secrets

**Get your private key:**
```bash
# View the private key (copy manually)
cat ~/.ssh/id_ed25519

# Or copy directly to clipboard (macOS)
cat ~/.ssh/id_ed25519 | pbcopy

# If using custom filename
cat ~/.ssh/id_ed25519_dreamhost | pbcopy
```

**Important:** Copy the **entire file contents**, including:
- The `-----BEGIN OPENSSH PRIVATE KEY-----` line
- All the base64-encoded content
- The `-----END OPENSSH PRIVATE KEY-----` line
- The trailing newline is normal and should be kept

**In GitHub:**
1. Go to your repository → Settings → Secrets and variables → Actions → New repository secret
2. Add the following secrets:
   - `SSH_PRIVATE_KEY`: Paste the **entire private key** from above (including BEGIN/END lines)
   - `SERVER_IP`: Your Dreamhost server address (e.g., `your-server.dreamhost.com`)
   - `SFTP_USERNAME`: Your Dreamhost username
   - `SFTP_DESTINATION_PATH`: The destination path for files (e.g., `/home/username/derbynet.org`)

### 5. Server Configuration for DerbyNet

DerbyNet requires write access for:
- **SQLite database** - `local/` directory (supports custom filenames)
- **Photo uploads** - `photos/` directory with subdirectories for racer/car photos

The deployment workflow (`.github/workflows/deploy.yml`) automatically creates these directories and sets proper permissions after each deployment.

**Note:** The `local/` directory is excluded from deployment to preserve your production database.

### 6. Deploy via GitHub Actions

Once the secrets are configured, your GitHub Actions workflow will automatically:
1. Build the DerbyNet application
2. Deploy files to Dreamhost via SFTP
3. Set up proper file permissions for SQLite and photos

Push to the `main` branch to trigger deployment, or use the "Actions" tab in GitHub to manually trigger the workflow.

## License

MIT License - see [MIT-LICENSE.txt](MIT-LICENSE.txt)
