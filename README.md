# Nano11Builder

A script to make your OWN Nano11 image.

This is a script to automate the build of an Nano 11 iso.
The main goal of this is to use only Microsoft utilities like DISM, and nothing external. The only executable included is oscdimg.exe, which is provided in the Windows ADK and it is used to create bootable ISO images. Also included is an unattended answer file, which is used to bypass the MS account on OOBE.

To download the post-setup, you will need git If you don't want to install it, you can skip the step

Instructions:

1. Download an Windows 11 ISO.
2. Mount the downloaded ISO image using Windows Explorer.
3. Run the nano11builder.bat file.
4. Select the drive letter where the image is mounted (only the letter, no colon (:))
5. Select the SKU that you want the image to be based.
6. Sit back and relax :)
7. When the image is completed, you will see it in the folder where the script was extracted, with the name nano11.iso

What is removed:

Everything Nano11 Removes

## Building in GitHub Actions

**Yes, you can create Windows 11 ISOs in GitHub Actions!** 

GitHub Actions provides Windows Server runners that include all necessary tools (DISM, PowerShell, etc.) to build Nano11 ISOs in the cloud.

👉 **[See detailed GitHub Actions guide](GITHUB_ACTIONS.md)** for complete instructions.

### Quick Start:

1. Go to the **Actions** tab in your GitHub repository
2. Select the **"Build Nano11 ISO"** workflow
3. Click **"Run workflow"**
4. Configure the workflow:
   - **ISO URL**: Provide a direct download link to a Windows 11 ISO
   - **Architecture**: Choose x64 or arm64
   - **Edition**: Choose standard or copilot edition
   - **Image Index**: Select the Windows edition (usually 1 for Home, 6 for Pro)
5. Click **"Run workflow"** button
6. Wait for the build to complete (typically 20-40 minutes)
7. Download the resulting ISO from the **Artifacts** section

### Key Benefits:

- ✓ No local Windows machine needed
- ✓ Automated and reproducible builds
- ✓ DISM and other tools pre-installed
- ✓ Build logs available for troubleshooting
- ✓ Can build both x64 and ARM64 versions

### Limitations:

- GitHub Actions artifact size limit: 10GB per artifact
- Artifact retention: 7 days by default (configurable)
- Requires a valid Windows 11 ISO URL for download
- Free GitHub accounts have monthly Actions minutes limits

### Running without an ISO URL:

You can run the workflow without providing an ISO URL to verify that your environment is correctly configured. The workflow will display system information and confirm that all tools are available.

ARM64 Support:

ARM64 versions of the scripts are now available:
- `Nano11Builder_arm64.bat` - ARM64 version of the main script
- `nano11builder_copilot_arm64.bat` - ARM64 version of the Copilot edition

These scripts are adapted to work with Windows 11 ARM64 images by:
- Using arm64 architecture in package names instead of amd64
- Removing wow64 (32-bit compatibility) references not applicable to ARM64
- Adjusting paths for ARM64 architecture (e.g., no "Program Files (x86)" folder)

To use the ARM64 scripts:
1. Download a Windows 11 ARM64 ISO
2. Mount the ISO
3. Run either `Nano11Builder_arm64.bat` or `nano11builder_copilot_arm64.bat`
4. Follow the same instructions as the x64 version

Known issues:

1. Only en-us language is supported as of now. This can be easily fixable by the end user, just by replacing every instance of en-us with the language needed (like ro-RO and so on).

And that's pretty much it for now!
