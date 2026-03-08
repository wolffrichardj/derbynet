# Running DerbyNet on Dreamhost

This guide covers Dreamhost-specific configuration and setup for DerbyNet.

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

The deployment workflow automatically creates this configuration. Simply update the `DERBYNET_URL` in `.github/workflows/deploy.yml`:

```yaml
env:
  DEPLOY_PATH: ${{ secrets.SFTP_DESTINATION_PATH }}/derbynet
  DERBYNET_URL: "https://yourdomain.com/derbynet"  # Change this to your domain
```

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

## Deployment

See `README.md` section "Deployment to Dreamhost" for:
- SSH key setup
- GitHub Actions configuration
- Automated deployment process

## Support

For general DerbyNet documentation, see:
- `README.md` - Project overview
- `ARCHITECTURE.md` - System architecture
- `DEVELOPMENT.md` - Development setup
- `docs/` - User documentation (PDF files)
