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

During setup, DerbyNet will prompt for photo directory paths. Use relative paths:

- **Racer photos**: `photos/head`
- **Car photos**: `photos/car`

These directories are automatically created with proper permissions by the deployment workflow.

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
