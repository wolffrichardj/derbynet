# Running DerbyNet on Dreamhost

This guide covers Dreamhost-specific configuration and setup for DerbyNet.

## Deployment Setup

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
   - `SFTP_DESTINATION_PATH`: The full destination path for files (e.g., `/home/username/derbynet.org` or `/home/username/domain.com/derbynet`). Note: The workflow deploys directly to this path.
   - `DERBYNET_URL`: The public URL for your DerbyNet installation (e.g., `https://yourdomain.com/derbynet`). This is used for mobile check-in QR codes.

### 5. Deploy via GitHub Actions

Once the secrets are configured, your GitHub Actions workflow will automatically:
1. Build the DerbyNet application
2. Deploy files to Dreamhost via SFTP
3. Set up proper file permissions for SQLite and photos

Push to the `main` branch to trigger deployment, or use the "Actions" tab in GitHub to manually trigger the workflow.

## Initial Setup

After deploying DerbyNet to your Dreamhost server, navigate to your site (e.g., `https://yourdomain.com/derbynet/`) and you'll see the setup page.

### Database Configuration

**Important:** Use the **SQLite data source** option with a relative path.

1. Click **"Choose Database"**
2. Select **"SQLite data source"**
3. In the "Path (on server) to SQLite data file:" field, enter:
   ```
   local/derbynet.sqlite
   ```
   Or use the **Browse** button to navigate to the `local/` directory

4. The connection string should show:
   ```
   sqlite:local/derbynet.sqlite
   ```

5. Click **Submit**

**Why relative paths?**
- The deployment automatically creates and sets permissions on the `local/` directory
- Using a relative path like `local/derbynet.sqlite` ensures the database is created in the correct location
- Absolute paths (starting with `/`) may point to system directories where the web server lacks write permissions

### Photo Directories

During setup, DerbyNet will prompt for photo directory paths. **Important:** The fields may show errors initially - this is expected.

**Enter these relative paths:**

1. **Directory for racer photos:**
   ```
   photos/head
   ```

2. **Directory for car photos:**
   ```
   photos/car
   ```

3. **Directory for videos** (if enabled):
   ```
   photos/videos
   ```

**Note:** You may see red warning messages saying "Directory does not exist or is not readable" - this is normal during initial setup. The directories exist and have proper permissions (set by deployment), but DerbyNet's validation may not recognize them until after you submit the form.

**After submitting:**
- The directories will be recognized
- DerbyNet will create subdirectories as needed (e.g., `200x200/`, `cropped/`)
- Photo uploads will work correctly

These directories are automatically created with proper permissions (`777`) by the deployment workflow.

### Custom Domain for Mobile Check-In

By default, DerbyNet uses the server's IP address for mobile check-in QR codes. To use your domain name instead:

**Automated (Recommended):**

The deployment workflow automatically creates this configuration using the `DERBYNET_URL` secret. Ensure you have added `DERBYNET_URL` to your GitHub repository secrets.

The next deployment will automatically create/update the `local/config-url` file.

**Manual Setup:**

If you need to set this without redeploying:

1. **SSH into your Dreamhost server:**
   ```bash
   ssh username@your-server.dreamhost.com
   cd /path/to/derbynet/local
   ```

2. **Create a `config-url` file:**
   ```bash
   echo "https://yourdomain.com/derbynet" > config-url
   ```

**After configuration:**
- Mobile check-in QR codes will use your domain name
- The URL will be `https://yourdomain.com/derbynet/mcheckin.php`
- This persists across deployments (the `local/` directory is excluded from deployment, but the config-url file is recreated each time)

## Security & Access Control

**Initial setup:** No login required - DerbyNet automatically grants setup permissions when no database exists.

**After setup:** Authentication is enforced with role-based permissions.

For detailed information on user roles, permissions, and security configuration, see the DerbyNet documentation in the `docs/` directory.

## Advanced Settings

Some DerbyNet features require access to advanced settings during setup:

- **Database path configuration** - Requires choosing the correct relative path
- **Photo directory configuration** - Specify where uploaded photos are stored
- **Lane count and other race settings** - Configure based on your track

Make sure to review all settings during initial setup.

## Troubleshooting

### Database Creation Fails

**Symptom:** Error when trying to initialize the database

**Common causes:**
1. **Incorrect path** - Using an absolute path like `/20260111-0759.sqlite` instead of `local/derbynet.sqlite`
2. **Permission issues** - The `local/` directory needs `777` permissions (set automatically by deployment)

**Solution:**
- Use relative path: `local/derbynet.sqlite`
- Verify permissions via SSH:
  ```bash
  cd /path/to/derbynet
  ls -la local/
  # Should show drwxrwxrwx (777)
  ```

### Photo Upload Fails

**Symptom:** Cannot upload racer or car photos

**Solution:**
- Verify photo directories exist and have proper permissions:
  ```bash
  cd /path/to/derbynet
  ls -la photos/
  # Should show drwxrwxrwx (777) for photos, head, and car directories
  ```

### After Deployment, Database is Missing

**Expected behavior:** The `local/` directory is **excluded** from deployment to preserve your production database.

- First deployment: You'll need to run setup to create the database
- Subsequent deployments: Your database persists and is not overwritten

## File Permissions

The deployment workflow automatically sets these permissions:

- `local/` directory: `777` (web server can create/write database)
- `photos/` directory: `777` (web server can upload/manage photos)

If you manually create these directories, ensure they have `777` permissions:

```bash
chmod -R 777 local
chmod -R 777 photos
```

## Support

For general DerbyNet documentation, see:
- `README.md` - Project overview
- `ARCHITECTURE.md` - System architecture
- `DEVELOPMENT.md` - Development setup
- `docs/` - User documentation (PDF files)
