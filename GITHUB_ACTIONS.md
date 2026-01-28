# GitHub Actions Build Guide

This document explains how to build Nano11 ISOs using GitHub Actions.

## Quick Answer

**Q: Can we create Windows 11 ISOs in GitHub Actions, or do we need a Windows machine?**

**A: Yes! You can build Nano11 ISOs entirely in GitHub Actions without a local Windows machine.**

## Why GitHub Actions Works

GitHub Actions provides:

- **Windows Server Runners**: Pre-configured with Windows OS
- **DISM Tool**: Already installed for image manipulation
- **PowerShell & Batch**: Full scripting support
- **Sufficient Storage**: ~14GB free space on runners
- **Artifact Uploads**: Store and download your built ISOs

## How to Build

### Step 1: Navigate to Actions Tab

1. Go to your repository on GitHub
2. Click on the **Actions** tab at the top
3. Find the **"Build Nano11 ISO"** workflow in the left sidebar

### Step 2: Start a Workflow Run

1. Click on **"Build Nano11 ISO"**
2. Click the **"Run workflow"** dropdown button (on the right)
3. You'll see a form with the following options:

### Step 3: Configure Your Build

**Required Inputs:**

- **ISO URL** (Optional): Direct download link to Windows 11 ISO
  - Leave empty for a test run (no actual ISO will be built)
  - For actual builds, provide a valid download URL
  - Example format: `https://example.com/Win11_English_x64.iso`

- **Architecture**: Choose your target architecture
  - `x64` - For standard PCs (default)
  - `arm64` - For ARM-based devices

- **Edition**: Select the build variant
  - `standard` - Standard Nano11 build
  - `copilot` - Nano11 with Copilot customizations

- **Image Index**: Windows edition to use (default: 1)
  - `1` - Windows Home
  - `6` - Windows Pro
  - Other indices for different editions

### Step 4: Run and Monitor

1. Click **"Run workflow"** button (green button at bottom of form)
2. The workflow will appear in the list below
3. Click on the workflow run to see real-time logs
4. Build typically takes 20-40 minutes depending on:
   - ISO download size and speed
   - Build complexity
   - Runner availability

### Step 5: Download Your ISO

Once the build completes:

1. Scroll down to the **"Artifacts"** section at the bottom of the workflow run page
2. You'll see an artifact named like: `nano11-x64-standard`
3. Click to download (it will be zipped)
4. Extract and use your Nano11 ISO!

## Test Run (No ISO Required)

You can verify the workflow works without providing an ISO URL:

1. Run the workflow with the ISO URL field **empty**
2. The workflow will:
   - Show system information
   - Verify DISM is available
   - Display available disk space
   - Confirm the environment is ready
3. This is useful for testing before doing a full build

## Workflow Process

Here's what happens during a build:

```
1. Checkout Code
   ↓
2. Check System (Windows version, DISM, disk space)
   ↓
3. Generate ISO cache key (hash of ISO URL)
   ↓
4. Restore ISO from cache (if available)
   ↓
5. Download Windows 11 ISO (only if not cached)
   ↓
6. Verify ISO file exists and has correct size
   ↓
7. Mount ISO to a drive letter
   ↓
8. Select appropriate build script (based on architecture/edition)
   ↓
9. Create automated version of script (removes interactive prompts)
   ↓
10. Run Nano11 Builder script
    - Copy files
    - Remove bloatware
    - Apply registry tweaks
    - Create new ISO
   ↓
11. Upload ISO as artifact
   ↓
12. Cleanup temporary files (preserves source ISO for caching)
   ↓
13. Display summary
```

## ISO Caching

The workflow automatically caches downloaded ISOs to speed up subsequent builds:

### How It Works

- **Cache Key**: Generated from a hash of the ISO URL
- **Cache Storage**: Uses GitHub Actions cache (10GB limit per repository)
- **Cache Duration**: Automatically evicted after 7 days of inactivity
- **Automatic**: No configuration needed

### Benefits

- **Time Savings**: Skip ~6GB download on repeat builds (saves 5-10 minutes)
- **Bandwidth Savings**: Reduces data transfer for both GitHub and users
- **Reliability**: Fewer network-related failures

### Cache Behavior

- **First Build**: Downloads ISO and caches it
- **Subsequent Builds**: Restores ISO from cache instantly
- **Different ISOs**: Each unique ISO URL has its own cache entry
- **Same ISO, Different Build**: Cache is reused across different architectures/editions

### Example Timeline

| Build | ISO URL | Download Time | Cache Status |
|-------|---------|---------------|--------------|
| Build #1 | URL-A | 8 minutes | Cached |
| Build #2 | URL-A | 5 seconds | Cache hit |
| Build #3 | URL-B | 8 minutes | Cached |
| Build #4 | URL-A | 5 seconds | Cache hit |

### Manual Cache Management

To clear the cache:
1. Go to **Settings** → **Actions** → **Caches**
2. Find caches starting with `windows11-iso-`
3. Click the delete icon to remove specific cache entries

This is useful if you need to re-download a corrupted ISO or free up cache space.

**Note on Cache Eviction**: Caches are automatically evicted:
- After 7 days of no access (not use) to the cache entry
- When the repository exceeds 10GB total cache size (oldest entries removed first)
- Can be manually deleted at any time

## Limitations & Considerations

### GitHub Actions Limits

- **Artifact Size**: Maximum 10GB per artifact
  - Nano11 ISOs are typically 3-5GB, well within limits
  
- **Artifact Retention**: Default 7 days
  - Can be configured up to 90 days in repository settings
  
- **Disk Space**: ~14GB available on Windows runners
  - Sufficient for ISO download + build process
  
- **Build Time**: Free accounts have monthly limits
  - 2,000 minutes/month for free accounts
  - Typical build: 30 minutes = ~66 builds/month

### ISO Download

- You need a direct download URL for Windows 11 ISO
- The URL must be publicly accessible
- First download counts toward workflow execution time
- Downloaded ISOs are automatically cached for future builds
- Cache limit: 10GB total per repository (sufficient for 1-2 ISOs)
- Consider using a fast CDN for quicker initial downloads

### Security Notes

- ISO URLs are logged in workflow runs
- Don't use this for sensitive/proprietary ISOs
- Built ISOs are temporarily stored as artifacts
- Artifacts are only accessible to repository collaborators

## Troubleshooting

### Build Fails During ISO Download

- Verify the ISO URL is correct and accessible
- Check if the URL requires authentication (not supported)
- Try a different ISO source

### "Not enough disk space" Error

- Windows 11 ISOs are 4-6GB
- Build process needs ~10-12GB total
- This shouldn't happen on standard runners
- If it does, report to GitHub Support

### Script Fails During Build

- Check the logs for specific error messages
- Verify the image index is correct
- Ensure the ISO is a valid Windows 11 installation media
- Some heavily customized ISOs may not work

### Artifact Upload Fails

- Check if the ISO was created (look for .iso files in logs)
- Verify the ISO isn't larger than 10GB
- Check repository storage quota

## Advanced Usage

### Matrix Builds

You can modify the workflow to build multiple configurations in parallel:

```yaml
strategy:
  matrix:
    architecture: [x64, arm64]
    edition: [standard, copilot]
```

### Scheduled Builds

Add a schedule trigger to build automatically:

```yaml
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sundays
```

### Custom Scripts

The workflow can be modified to:
- Use different ISO sources
- Apply additional customizations
- Upload to cloud storage instead of artifacts
- Send notifications on completion

## Cost Comparison

| Method | Cost | Time | Setup |
|--------|------|------|-------|
| Local Windows PC | Hardware cost | 20-30 min | Manual setup |
| GitHub Actions (Free) | $0 | 30-40 min | One-time workflow |
| GitHub Actions (Paid) | ~$0.15/build | 30-40 min | One-time workflow |

*Note: Times are approximate and depend on hardware/network*

## Examples

### Building Standard x64 Edition

1. Architecture: `x64`
2. Edition: `standard`
3. Image Index: `6` (for Pro)
4. ISO URL: `https://example.com/Win11_Pro_x64.iso`

Result: `nano11.iso` (~3.5GB)

### Building ARM64 Copilot Edition

1. Architecture: `arm64`
2. Edition: `copilot`
3. Image Index: `1` (for Home)
4. ISO URL: `https://example.com/Win11_Home_ARM64.iso`

Result: `nano11_copilot_arm64.iso` (~3.2GB)

## Summary

GitHub Actions makes it possible to build Nano11 ISOs without any local Windows machine. The workflow is:

✅ **Automated** - No manual intervention required  
✅ **Reproducible** - Same inputs = same outputs  
✅ **Free-tier friendly** - Works with free GitHub accounts  
✅ **Well-documented** - Build logs show every step  
✅ **Accessible** - Download from anywhere  

This answers the original question: **You do NOT need a Windows machine - GitHub Actions provides everything you need!**
