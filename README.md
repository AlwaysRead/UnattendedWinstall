# `autounattend.xml` - Unattended Windows Installation Configuration

## Overview

The `autounattend.xml` file is an **answer file** used to automate the installation of the Windows operating system. It allows predefined configurations for system settings, disk partitions, user accounts, and custom scripts to streamline deployment across multiple machines, ensuring consistency and reducing manual effort.

This specific `autounattend.xml` file is tailored for Windows 10/11 installations and is designed to achieve the following:

- Automate disk partitioning and formatting.
- Bypass certain hardware checks (e.g., TPM, Secure Boot, RAM).
- Configure language and region settings for English (India) and English (United States).
- Create a local administrator account named **Admin**.
- Execute custom PowerShell scripts for system debloating and optimization.
- Customize system appearance, including the Start menu and taskbar.
- Apply registry changes and disable features like Windows Copilot.

## Structure and Functionality

The file is divided into **configuration passes**, each executed at a specific phase of the installation process.

### 1. `windowsPE` Pass

**Purpose:** Configures initial setup during the Windows Preinstallation Environment.

**Key Actions:**

- Sets system locale: `en-IN` and UI locale: `en-US`.
- Applies a generic product key: `BT79Q-G7N6G-PGBYW-4YWX6-6F4BT` (used for image deployment).
- Configures disk layout:
  - EFI System Partition (300 MB, FAT32)
  - Microsoft Reserved Partition (16 MB)
  - Primary NTFS Partition for Windows
  - NTFS Recovery Partition with GPT attributes
- Bypasses hardware requirements by modifying registry (`HKLM\SYSTEM\Setup\LabConfig`):
  - Disables TPM, Secure Boot, and RAM checks.

### 2. `specialize` Pass

**Purpose:** Applies configurations after the Windows image is applied, but before first user logon.

**Key Actions:**

- Extracts and runs embedded PowerShell scripts.
- Loads and modifies the default user profile registry.
- Removes unnecessary built-in applications and Windows features:
  - Example: Cortana, Xbox Apps, Internet Explorer.
- Configures system appearance:
  - Applies dark theme
  - Sets custom accent color (`#0078D4`)
- Disables or configures features:
  - Disables OneDrive, Widgets, Windows Copilot.
  - Sets group policy and registry keys (e.g., disables BitLocker).

### 3. `oobeSystem` Pass

**Purpose:** Finalizes the installation during the Out-of-Box Experience (OOBE).

**Key Actions:**

- Creates a local administrator account named `Admin` (no password by default).
- Skips user and device setup screens to expedite deployment.
- Executes final personalization scripts:
  - Sets default folder view to Desktop.
  - Disables File Explorer grouping and search box.
  - Applies theme and restarts Explorer to reflect UI changes.
- Removes `C:\Windows.old` and clears autologon settings.

## 4. Custom PowerShell Scripts

This setup includes multiple embedded PowerShell scripts to automate debloating and optimization.

### Included Scripts:

| Script | Description |
|--------|-------------|
| `ExtractFiles.ps1` | Extracts embedded files (e.g., `.ps1`, `.reg`, `.xml`) to disk during setup. |
| `RemovePackages.ps1` | Removes pre-installed apps (e.g., Xbox, Cortana, 3D Viewer). |
| `RemoveCapabilities.ps1` | Removes optional capabilities (e.g., Internet Explorer, WMP). |
| `RemoveFeatures.ps1` | Uninstalls optional features (e.g., PowerShell v2, RDP client). |
| `MakeEdgeUninstallable.ps1` | Changes group policy to allow uninstalling Microsoft Edge. |
| `SetStartPins.ps1` | Clears pinned apps from the Start menu. |
| `SetColorTheme.ps1` | Sets a dark system theme and applies the custom accent color. |
| `DefaultUser.ps1` | Applies user defaults: hides file extensions, disables Copilot, etc. |
| `UserOnce.ps1` | Executes one-time user-specific configurations at first logon. |
| `FirstLogon.ps1` | Finalizes installation by cleaning up old files and logs. |

> **Note:** These scripts are essential for achieving a lightweight, optimized Windows environment. Review and test each script before deployment to ensure it aligns with your organizational or personal setup needs.

## Usage Instructions

### 1. Prepare Installation Media

- Download a Windows ISO (Windows 10/11).
- Mount the ISO and copy contents to a USB or folder.
- Place `autounattend.xml` in the **root directory** of the media.

### 2. Boot and Install

- Boot the target machine from the prepared media.
- Ensure UEFI is enabled (required for GPT partitioning).
- Installation will begin automatically using the `autounattend.xml` configuration.

### 3. Post-Installation

- Log in using the **Admin** account (no password).
- Verify changes:
  - Review logs in `C:\Windows\Setup\Scripts` (`Specialize.log`, `FirstLogon.log`).
  - Confirm removed applications, UI changes, and registry settings.

## Key Considerations

- **Product Key:** Replace the generic key with a valid license key for your Windows version.
- **Data Loss Warning:** This setup wipes the entire disk (Disk 0). Backup any important data before installation.
- **Hardware Bypass:** This configuration bypasses TPM, RAM, and Secure Boot checks for compatibility on older hardware.
- **Script Execution:** Ensure the `C:\Windows\Setup\Scripts` directory is writable and not blocked by group policies.
- **Language Settings:** Default locale settings are `en-IN` and `en-US`. Modify the `<InputLocale>`, `<SystemLocale>`, `<UILanguage>`, and `<UserLocale>` elements as needed.
- **Security Warning:** The Admin account has no password. For production environments, either add a password or enable password prompt during OOBE.
- **Customization:** Review all embedded PowerShell scripts to ensure the configuration aligns with your target use case.

## Troubleshooting

| Issue | Resolution |
|-------|------------|
| **Installation fails** | Check `X:\diskpart.log` for partitioning issues. Ensure the target disk is detected and unlocked. |
| **Scripts don’t run** | Review logs in `C:\Windows\Setup\Scripts`. Validate file paths and script permissions. |
| **Regional settings incorrect** | Verify `<Component>` entries in `windowsPE` and `oobeSystem` passes. |
| **Apps/features not removed** | Ensure your Windows version matches the script selectors and app names. |

## License

This `autounattend.xml` file and associated scripts are provided **as-is**, without warranty. Use at your own risk. Ensure compliance with Microsoft's licensing terms and software usage policies.

## Contact

For assistance or additional documentation, refer to:

- [Microsoft Docs - Windows Unattended Setup Reference](https://docs.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/)
