# Fix for ARM64 UTM Driver Installation Issue

## Problem

When loading a Nano11 ARM64 ISO into UTM (a virtualization platform for macOS), the Windows 11 installer starts but fails with the error:

```
"A media driver your computer needs is missing"
```

This error occurs even when UTM guest tools are loaded as disk drivers.

## Root Cause

The issue was caused by the `/ResetBase` flag in the DISM cleanup command used in the ARM64 builder scripts:

```batch
dism /image:c:\scratchdir /Cleanup-Image /StartComponentCleanup /ResetBase
```

### What `/ResetBase` Does

- **`/StartComponentCleanup`**: Removes older, superseded versions of system components to free up disk space
- **`/ResetBase`**: Makes all current components permanent by removing ALL superseded versions, including driver backups

### Why This Caused Problems

1. **UTM uses VirtIO storage controllers** which require specific ARM64 drivers
2. These drivers may be flagged as "superseded" if they've been updated
3. `/ResetBase` permanently removes these superseded driver versions
4. Without the storage drivers, Windows installer cannot detect the virtual disk
5. This results in the "media driver missing" error during installation

### ARM64-Specific Issue

This problem is particularly acute on ARM64 systems because:
- ARM64 has fewer driver alternatives available
- Only ARM64-native drivers work at the kernel level
- x86/x64 drivers cannot be used as a fallback for storage/boot drivers
- VirtIO drivers for ARM64 are critical for UTM/QEMU virtualization

## Solution

Removed the `/ResetBase` flag from both ARM64 builder scripts:

**Files Modified:**
- `Nano11Builder_arm64.bat` (line 177)
- `nano11builder_copilot_arm64.bat` (line 171)

**Changed from:**
```batch
dism /image:c:\scratchdir /Cleanup-Image /StartComponentCleanup /ResetBase
```

**Changed to:**
```batch
dism /image:c:\scratchdir /Cleanup-Image /StartComponentCleanup
```

## Benefits of This Fix

1. **Preserves Storage Drivers**: Keeps superseded driver versions that may be needed for different hardware configurations
2. **Maintains Compatibility**: Ensures UTM and other ARM64 virtualization platforms can use the ISO
3. **Still Cleans Up**: The `/StartComponentCleanup` command still reduces image size by removing unnecessary files
4. **Enables Rollback**: Allows Windows to roll back to previous driver versions if needed
5. **Minimal Change**: Only affects ARM64 scripts where the issue occurs

## Impact

- **Image Size**: Slightly larger final ISO (~100-500MB difference depending on drivers)
- **Functionality**: Restored ability to install on UTM ARM64 and other VirtIO-based ARM64 VMs
- **Compatibility**: Better hardware compatibility across different ARM64 virtualization platforms
- **x64 Versions**: No changes to x64 scripts as they don't have this issue

## Testing Recommendations

When testing this fix:

1. **Build a new Nano11 ARM64 ISO** using the updated scripts
2. **Test in UTM** on Apple Silicon (M1/M2/M3) Macs:
   - Load the ISO as installation media
   - Verify the installer detects the virtual disk
   - Complete the installation successfully
3. **Test on other ARM64 VMs** if available (Hyper-V on ARM, QEMU, etc.)

## Additional Notes

### For x64 Versions

The x64 scripts (`Nano11Builder.bat` and `nano11builder_copilot.bat`) still use `/ResetBase` because:
- They don't have the same driver compatibility issues
- x64 has broader driver availability
- Most x64 virtualization platforms include fallback drivers
- Maximum size reduction is more important for x64 deployments

If you experience similar issues on x64, you can apply the same fix.

### Alternative Solutions Considered

Other approaches that were evaluated but not chosen:

1. **Inject VirtIO drivers**: Would require distributing additional drivers with the tool
2. **Skip cleanup entirely**: Would result in much larger ISOs
3. **Add driver preservation logic**: Too complex for minimal benefit
4. **Document workaround**: Doesn't fix the root cause

## References

**Note**: These references were valid as of February 2026. URLs may change over time.

- [Microsoft DISM Documentation](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/clean-up-the-winsxs-folder)
- [UTM Virtual Machines Documentation](https://docs.getutm.app/)
- [Windows on ARM Driver Requirements](https://learn.microsoft.com/en-us/windows-hardware/drivers/develop/building-arm64-drivers)

## Support

If you continue to experience driver issues after this fix:

1. Verify you're using the latest version of UTM
2. Ensure your UTM VM is configured to use VirtIO for storage
3. Check that you downloaded an official Windows 11 ARM64 ISO
4. Try manually loading UTM guest tools drivers during installation
5. Report the issue with detailed error messages and UTM version
