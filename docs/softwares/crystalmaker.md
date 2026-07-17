# CrystalMaker 11.7.3 on Fedora 44 with WineHQ Development

**Author:** Zixian Tan  
**Last updated:** July 2026  
**Tested Wine version:** WineHQ Development 11.13  
**Tested desktop:** GNOME on Wayland  
**Tested GPU:** NVIDIA GeForce RTX 3080

> [!IMPORTANT]
> This guide records the final configuration that worked after clean-prefix
> testing. It deliberately excludes unsuccessful Bottles, Proton, DXVK, and
> VKD3D experiments from the main installation path.

> [!NOTE]
> CrystalMaker and Microsoft fonts are proprietary. Obtain CrystalMaker, a
> valid licence, and any Windows font files from legitimate sources. Do not
> redistribute installers, licence codes, or font files with this guide.

## Contents

- [CrystalMaker 11.7.3 on Fedora 44 with WineHQ Development](#crystalmaker-1173-on-fedora-44-with-winehq-development)
	- [Contents](#contents)
	- [1. What this guide installs](#1-what-this-guide-installs)
	- [2. Tested configuration](#2-tested-configuration)
	- [3. Conventions and paths](#3-conventions-and-paths)
	- [4. Prepare Fedora](#4-prepare-fedora)
	- [5. Install WineHQ Development](#5-install-winehq-development)
	- [6. Create a clean 64-bit Wine prefix](#6-create-a-clean-64-bit-wine-prefix)
		- [6.1 Clear unrelated test variables](#61-clear-unrelated-test-variables)
		- [6.2 Select a new prefix path](#62-select-a-new-prefix-path)
		- [6.3 Initialise the prefix](#63-initialise-the-prefix)
	- [7. Configure Windows 10 and the OpenGL backend](#7-configure-windows-10-and-the-opengl-backend)
		- [7.1 Select Windows 10](#71-select-windows-10)
		- [7.2 Force Wine to use GLX instead of EGL](#72-force-wine-to-use-glx-instead-of-egl)
	- [8. Install basic fonts with Winetricks](#8-install-basic-fonts-with-winetricks)
	- [9. Install Microsoft .NET Desktop Runtime 8](#9-install-microsoft-net-desktop-runtime-8)
	- [10. Install CrystalMaker](#10-install-crystalmaker)
	- [11. Configure WPF and OpenGL](#11-configure-wpf-and-opengl)
		- [11.1 Keep the GLX setting](#111-keep-the-glx-setting)
		- [11.2 Disable WPF hardware acceleration](#112-disable-wpf-hardware-acceleration)
	- [12. Confirm DXVK and VKD3D are disabled](#12-confirm-dxvk-and-vkd3d-are-disabled)
	- [13. Perform the first functional test](#13-perform-the-first-functional-test)
	- [14. Make the installation per-user for group licences](#14-make-the-installation-per-user-for-group-licences)
		- [14.1 Symptom](#141-symptom)
		- [14.2 Identify the Wine username](#142-identify-the-wine-username)
		- [14.3 Copy the complete application directory](#143-copy-the-complete-application-directory)
		- [14.4 Why changing MSI properties after installation was insufficient](#144-why-changing-msi-properties-after-installation-was-insufficient)
	- [15. Install and register Segoe UI fonts](#15-install-and-register-segoe-ui-fonts)
		- [15.1 Obtain the font files legally](#151-obtain-the-font-files-legally)
		- [15.2 Register Segoe UI in Wine](#152-register-segoe-ui-in-wine)
		- [15.3 Remove an incorrect substitution](#153-remove-an-incorrect-substitution)
		- [15.4 Refresh and verify fonts](#154-refresh-and-verify-fonts)
	- [16. Create a native GNOME launcher](#16-create-a-native-gnome-launcher)
		- [16.1 Create the wrapper directory](#161-create-the-wrapper-directory)
		- [16.2 Create the desktop entry](#162-create-the-desktop-entry)
	- [17. Verify the final installation](#17-verify-the-final-installation)
		- [17.1 Configuration checks](#171-configuration-checks)
		- [17.2 Functional checklist](#172-functional-checklist)
		- [17.3 Cold-start test](#173-cold-start-test)
	- [18. Back up the working prefix](#18-back-up-the-working-prefix)
	- [19. Troubleshooting](#19-troubleshooting)
		- [19.1 `wglCreateContext` or pixel-format failure](#191-wglcreatecontext-or-pixel-format-failure)
		- [19.2 CrystalMaker starts but freezes or crashes when opening a crystal](#192-crystalmaker-starts-but-freezes-or-crashes-when-opening-a-crystal)
		- [19.3 Black interface, partial redraw, or content appears after scrolling](#193-black-interface-partial-redraw-or-content-appears-after-scrolling)
		- [19.4 Gallery works on one monitor but a crystal fails on another](#194-gallery-works-on-one-monitor-but-a-crystal-fails-on-another)
		- [19.5 CrystalMaker immediately exits](#195-crystalmaker-immediately-exits)
		- [19.6 MSI installation fails](#196-msi-installation-fails)
		- [19.7 Licence says it is valid only for a single-user installation](#197-licence-says-it-is-valid-only-for-a-single-user-installation)
		- [19.8 Square boxes replace punctuation or symbols](#198-square-boxes-replace-punctuation-or-symbols)
		- [19.9 Desktop launcher fails validation or disappears](#199-desktop-launcher-fails-validation-or-disappears)
		- [19.10 Collect a focused crash log](#1910-collect-a-focused-crash-log)
	- [20. Reverse-engineering notes](#20-reverse-engineering-notes)
	- [21. Registry reference](#21-registry-reference)
		- [OpenGL backend](#opengl-backend)
		- [WPF interface composition](#wpf-interface-composition)
		- [Wine font registration](#wine-font-registration)
		- [CrystalMaker installation classification](#crystalmaker-installation-classification)
	- [22. Maintenance and upgrades](#22-maintenance-and-upgrades)
		- [Update .NET 8 servicing releases](#update-net-8-servicing-releases)
		- [Test a newer Wine build](#test-a-newer-wine-build)
		- [Update CrystalMaker](#update-crystalmaker)
		- [Keep a configuration record](#keep-a-configuration-record)
	- [23. References](#23-references)
	- [Final known-good configuration](#final-known-good-configuration)

## 1. What this guide installs

The final configuration provides:

- CrystalMaker 11.7.3;
- Microsoft .NET Desktop Runtime 8 for Windows x64;
- WineHQ Development on Fedora 44;
- OpenGL rendering through Wine's GLX path;
- WPF software composition while retaining GPU-accelerated CrystalMaker OpenGL;
- working NVIDIA rendering;
- a per-user CrystalMaker location suitable for the tested group licence;
- Segoe UI and Segoe UI Symbol support;
- a native GNOME application launcher;
- repeatable diagnostics and backup procedures.

The successful graphics path is:

```text
CrystalMaker WPF interface -> WineD3D/software WPF composition
CrystalMaker 3D renderer    -> WGL/OpenGL -> GLX -> NVIDIA driver
```

The final configuration does **not** use DXVK or VKD3D.

## 2. Tested configuration

| Component | Tested value |
|---|---|
| Operating system | Fedora 44 Workstation |
| Desktop | GNOME Wayland |
| GPU | NVIDIA GeForce RTX 3080 |
| NVIDIA driver | 595.80 / 595.xx series |
| Wine | WineHQ Development 11.13 |
| Prefix architecture | win64 |
| Emulated Windows version | Windows 10 |
| CrystalMaker | 11.7.3 |
| Runtime | Microsoft Windows Desktop Runtime 8 x64 |
| Wine OpenGL backend | GLX (`UseEGL=N`) |
| WPF hardware acceleration | Disabled |
| Direct3D translation | WineD3D |
| Licence tested | Valid multi-seat/group licence |

Later Wine, driver, Fedora, and CrystalMaker releases may behave differently.
Test upgrades in a copied or new prefix before changing the working prefix.

## 3. Conventions and paths

Commands in this guide are written for Bash or Zsh.

Open a terminal and define the working prefix:

```bash
export WINEPREFIX="$HOME/.wine-crystalmaker"
export WINEARCH=win64
```

Confirm which Wine binaries the shell will use:

```bash
command -v wine
command -v wineserver
wine --version
```

The tested WineHQ Development package may also provide explicit binaries under:

```text
/opt/wine-devel/bin/wine
/opt/wine-devel/bin/wineserver
```

If several Wine builds are installed, use one build consistently for every
command affecting this prefix. For example:

```bash
export WINE_BIN="/opt/wine-devel/bin/wine"
export WINESERVER_BIN="/opt/wine-devel/bin/wineserver"

"$WINE_BIN" --version
```

If `wine` already reports the required WineHQ Development build, the shorter
commands used throughout this guide are sufficient.

Example installer locations used below:

```bash
export DOTNET_INSTALLER="$HOME/Downloads/windowsdesktop-runtime-8.0.x-win-x64.exe"
export CM_INSTALLER="$HOME/Downloads/crystalmaker11_win/WinCM.msi"
```

Replace these paths with the actual downloaded filenames.

## 4. Prepare Fedora

Update package metadata and the installed system:

```bash
sudo dnf upgrade --refresh
```

Install the supporting tools used by this guide:

```bash
sudo dnf install \
  cabextract \
  desktop-file-utils \
  file \
  fontconfig \
  mesa-demos \
  p7zip \
  p7zip-plugins \
  winetricks
```

If Fedora's `winetricks` package is unavailable or significantly outdated, use
the current release from the official Winetricks project. Do not keep a packaged
and manually installed Winetricks in the same command path.

Verify the native graphics stack before testing Wine:

```bash
glxinfo -B
```

For the tested NVIDIA system, the renderer should name the NVIDIA GPU rather
than a software renderer such as `llvmpipe`.

Useful focused checks are:

```bash
glxinfo | grep 'OpenGL renderer'
glxinfo | grep 'OpenGL version'
nvidia-smi
```

Wine does not need a Windows NVIDIA driver inside the prefix. Wine uses the
native Linux NVIDIA/OpenGL stack.

## 5. Install WineHQ Development

Add the WineHQ Fedora 44 repository using DNF5:

```bash
sudo dnf config-manager addrepo \
  --from-repofile=https://dl.winehq.org/wine-builds/fedora/44/winehq.repo
```

If the installed `config-manager` exposes the older syntax, use:

```bash
sudo dnf config-manager addrepo \
  https://dl.winehq.org/wine-builds/fedora/44/winehq.repo
```

Refresh metadata and install the development build:

```bash
sudo dnf makecache
sudo dnf install winehq-devel
```

Verify the selected Wine:

```bash
wine --version
rpm -q winehq-devel wine-devel
```

The configuration documented here was verified with:

```text
WineHQ Development 11.13
```

Do not assume that a different Wine binary found earlier in `PATH` is the same
build. If necessary, verify the explicit WineHQ binary:

```bash
/opt/wine-devel/bin/wine --version
```

## 6. Create a clean 64-bit Wine prefix

### 6.1 Clear unrelated test variables

Remove variables left by Bottles, Proton, DXVK, or GPU-offload experiments:

```bash
unset WINEPREFIX WINEARCH WINE WINEDEBUG WINESERVER

unset DXVK_HUD DXVK_LOG_LEVEL DXVK_FILTER_DEVICE_NAME DXVK_ASYNC

unset PROTON_USE_WINED3D
unset STEAM_COMPAT_DATA_PATH
unset STEAM_COMPAT_CLIENT_INSTALL_PATH

unset DRI_PRIME
unset __NV_PRIME_RENDER_OFFLOAD
unset __GLX_VENDOR_LIBRARY_NAME
unset __VK_LAYER_NV_optimus
unset VK_LOADER_DRIVERS_SELECT
```

### 6.2 Select a new prefix path

```bash
export WINEPREFIX="$HOME/.wine-crystalmaker"
export WINEARCH=win64
```

Protect an existing prefix from accidental replacement:

```bash
test ! -e "$WINEPREFIX" || {
  printf 'Refusing to overwrite existing prefix: %s\n' "$WINEPREFIX" >&2
  exit 1
}
```

If an existing prefix is valuable, close Wine and rename it instead of deleting
it:

```bash
wineserver -k
mv "$HOME/.wine-crystalmaker" \
   "$HOME/.wine-crystalmaker.backup-$(date +%Y%m%d-%H%M%S)"
```

Re-export the new path after renaming:

```bash
export WINEPREFIX="$HOME/.wine-crystalmaker"
export WINEARCH=win64
```

### 6.3 Initialise the prefix

```bash
wineboot -i
```

Wait until the Wine setup processes finish. Wine Mono may be skipped because
CrystalMaker uses Microsoft's official .NET Desktop Runtime. Gecko can be added
later if a web-based panel requires it.

Verify the prefix structure:

```bash
ls "$WINEPREFIX"
```

Expected entries include:

```text
dosdevices
drive_c
system.reg
user.reg
userdef.reg
```

Verify that the prefix contains a 64-bit Explorer:

```bash
file "$WINEPREFIX/drive_c/windows/explorer.exe"
```

The output should include `PE32+` and `x86-64`.

## 7. Configure Windows 10 and the OpenGL backend

### 7.1 Select Windows 10

Open Wine configuration:

```bash
winecfg
```

In the **Applications** tab, set **Windows Version** to **Windows 10**, apply the
change, and close the dialog.

Verify the registry value if required:

```bash
wine reg query 'HKCU\Software\Wine' /v Version
```

### 7.2 Force Wine to use GLX instead of EGL

The tested setup needed this registry value to avoid WGL/OpenGL context creation
failures under GNOME Wayland:

```bash
wine reg add \
  'HKCU\Software\Wine\X11 Driver' \
  /v UseEGL \
  /t REG_SZ \
  /d N \
  /f
```

Verify it:

```bash
wine reg query \
  'HKCU\Software\Wine\X11 Driver' \
  /v UseEGL
```

Expected result:

```text
UseEGL    REG_SZ    N
```

`UseEGL=N` is a Wine registry setting. The following is not an equivalent
replacement and should not be used:

```bash
export UseEGL=N
```

Restart the prefix so new processes read the value:

```bash
wineserver -k
wineboot -u
wineserver -k
```

This setting only needs to be written once per prefix.

## 8. Install basic fonts with Winetricks

Confirm that Winetricks uses the same prefix and Wine build:

```bash
export WINEPREFIX="$HOME/.wine-crystalmaker"
export WINE="$(command -v wine)"
export WINESERVER="$(command -v wineserver)"

winetricks --version
```

Install the tested basic font set:

```bash
winetricks -q corefonts tahoma
```

If the current Winetricks is installed in `~/.local/bin`, use:

```bash
"$HOME/.local/bin/winetricks" -q corefonts tahoma
```

Refresh the prefix:

```bash
wineserver -k
wineboot -u
wineserver -k
```

Check the installed basic fonts:

```bash
find "$WINEPREFIX/drive_c/windows/Fonts" \
  -maxdepth 1 -type f -printf '%f\n' \
  | sort \
  | grep -Ei 'arial|andale|cour|georgia|impact|tahoma|times|trebuc|verdana'
```

## 9. Install Microsoft .NET Desktop Runtime 8

CrystalMaker 11.7.3 is a .NET 8 WPF application. Install the **Windows x64 .NET
Desktop Runtime 8**, not only the console runtime and not the Linux runtime.

Download the current supported 8.0.x Windows x64 installer from Microsoft's
official .NET 8 download page.

Set the exact local installer path:

```bash
export DOTNET_INSTALLER="$HOME/Downloads/windowsdesktop-runtime-8.0.x-win-x64.exe"
```

Confirm it exists:

```bash
test -f "$DOTNET_INSTALLER" || {
  printf 'Installer not found: %s\n' "$DOTNET_INSTALLER" >&2
  exit 1
}
```

Run the installer:

```bash
wine "$DOTNET_INSTALLER"
```

After installation, inspect the shared runtime directory:

```bash
find \
  "$WINEPREFIX/drive_c/Program Files/dotnet/shared/Microsoft.WindowsDesktop.App" \
  -maxdepth 1 -mindepth 1 -type d -printf '%f\n'
```

An installed `8.0.x` directory should appear.

Also verify the base runtime:

```bash
find \
  "$WINEPREFIX/drive_c/Program Files/dotnet/shared/Microsoft.NETCore.App" \
  -maxdepth 1 -mindepth 1 -type d -printf '%f\n'
```

The Wine uninstaller should also list the runtime:

```bash
wine uninstaller
```

Restart Wine after the runtime installation:

```bash
wineserver -k
wineboot -u
wineserver -k
```

## 10. Install CrystalMaker

Use CrystalMaker's official MSI through the normal installer path. The clean
deployment test found this more reliable than an MSI administrative extraction.

Set and validate the installer path:

```bash
export CM_INSTALLER="$HOME/Downloads/crystalmaker11_win/WinCM.msi"

test -f "$CM_INSTALLER" || {
  printf 'Installer not found: %s\n' "$CM_INSTALLER" >&2
  exit 1
}
```

Run the MSI installer:

```bash
wine msiexec /i "$CM_INSTALLER"
```

To create a verbose MSI log in Zsh or Bash, quote `/L*V` so the shell does not
expand the asterisk:

```bash
wine msiexec /i \
  "$CM_INSTALLER" \
  '/L*V' \
  "$HOME/WinCM-install.log"
```

The normal installation path is:

```text
C:\Program Files\CrystalMaker Software\CrystalMaker
```

Verify the executable:

```bash
export CM_PROGRAM_FILES_EXE="$WINEPREFIX/drive_c/Program Files/CrystalMaker Software/CrystalMaker/WinCM.exe"

ls -lh "$CM_PROGRAM_FILES_EXE"
```

Do not activate the licence yet if it requires a single-user installation. The
per-user relocation is covered in Section 14.

## 11. Configure WPF and OpenGL

### 11.1 Keep the GLX setting

Confirm `UseEGL=N` remains present:

```bash
wine reg query \
  'HKCU\Software\Wine\X11 Driver' \
  /v UseEGL
```

### 11.2 Disable WPF hardware acceleration

CrystalMaker's main interface uses WPF. In the tested environment, disabling
WPF hardware acceleration avoided freezes or crashes when opening a crystal:

```bash
wine reg add \
  'HKCU\Software\Microsoft\Avalon.Graphics' \
  /v DisableHWAcceleration \
  /t REG_DWORD \
  /d 1 \
  /f
```

Verify it:

```bash
wine reg query \
  'HKCU\Software\Microsoft\Avalon.Graphics' \
  /v DisableHWAcceleration
```

Expected result:

```text
DisableHWAcceleration    REG_DWORD    0x1
```

This setting changes WPF interface composition. It does not disable
CrystalMaker's separate OpenGL crystal renderer or the native NVIDIA driver.

Restart the prefix:

```bash
wineserver -k
wineboot -u
wineserver -k
```

Both registry values normally persist and do not need to be rewritten before
each launch.

## 12. Confirm DXVK and VKD3D are disabled

DXVK caused black windows, incomplete dirty-region updates, and content that
appeared only after scrolling. VKD3D did not address the WPF D3D9/D3D9Ex path.

Do not install either component in the final prefix:

```text
Do not run: winetricks dxvk
Do not run: winetricks vkd3d
```

Inspect Wine DLL overrides:

```bash
wine reg query 'HKCU\Software\Wine\DllOverrides'
```

It is normal for a clean prefix not to have this key.

The following DLLs should not have `native` overrides from DXVK/VKD3D:

```text
d3d9
d3d10
d3d10_1
d3d10core
d3d11
d3d12
dxgi
```

If DXVK was previously installed in a disposable test prefix, prefer creating a
fresh prefix. If that is not practical, Winetricks may restore WineD3D:

```bash
winetricks --force wined3d
```

Then remove stale overrides only after inspecting them:

```bash
for dll in d3d9 d3d10 d3d10_1 d3d10core d3d11 d3d12 dxgi; do
  wine reg delete \
    'HKCU\Software\Wine\DllOverrides' \
    /v "$dll" \
    /f 2>/dev/null || true
done

wineserver -k
wineboot -u
wineserver -k
```

## 13. Perform the first functional test

Close all processes belonging to this prefix:

```bash
wineserver -k
```

Start the Program Files copy for the initial graphics test:

```bash
wine "$CM_PROGRAM_FILES_EXE"
```

Test in this order:

1. Confirm that the main window draws normally.
2. Open the Highlights or Gallery view.
3. Open the **Carbon - Diamond** example.
4. Confirm that a crystal appears.
5. Rotate, zoom, and pan the crystal.
6. Close the document.
7. Open a second example.
8. Exit CrystalMaker normally.

In another terminal, monitor the NVIDIA GPU:

```bash
watch -n 1 nvidia-smi
```

CrystalMaker should report an NVIDIA renderer and a working OpenGL mode. The
tested system reported OpenGL 4.0 in CrystalMaker while the native driver
reported OpenGL 4.6 compatibility.

For an OpenGL-focused log:

```bash
wineserver -k

WINEDEBUG=+opengl \
wine "$CM_PROGRAM_FILES_EXE" \
  &> "$HOME/crystalmaker-opengl.log"
```

Inspect relevant lines:

```bash
grep -nEi \
  'wglCreateContext|wglMakeCurrent|GL version|OpenGL context|renderer|failed' \
  "$HOME/crystalmaker-opengl.log" \
  | tail -100
```

## 14. Make the installation per-user for group licences

### 14.1 Symptom

A valid Education Group, Standard Group, or multi-seat licence may display:

```text
Invalid licence code

Sorry, your licence code is only valid for a single user installation.
```

In the tested CrystalMaker 11.7.3 build, group licence did not mean that the
application could be installed for all Windows users. The licence logic treated
an executable under `Program Files` as a per-machine installation.

### 14.2 Identify the Wine username

Run:

```bash
wine cmd /c 'echo %USERNAME%'
```

Set the returned value. In the tested prefix it matched the Linux account name:

```bash
export CM_WINE_USER="$USER"
```

Verify that the corresponding Wine user directory exists:

```bash
test -d "$WINEPREFIX/drive_c/users/$CM_WINE_USER" || {
  printf 'Wine user directory not found for: %s\n' "$CM_WINE_USER" >&2
  exit 1
}
```

### 14.3 Copy the complete application directory

Close Wine first:

```bash
wineserver -k
```

Define source and destination paths:

```bash
export CM_SOURCE_DIR="$WINEPREFIX/drive_c/Program Files/CrystalMaker Software/CrystalMaker"
export CM_DEST_PARENT="$WINEPREFIX/drive_c/users/$CM_WINE_USER/AppData/Local/Programs/CrystalMaker Software"
export CM_DEST_DIR="$CM_DEST_PARENT/CrystalMaker"
export CM_EXE="$CM_DEST_DIR/WinCM.exe"
```

Validate the source and protect an existing destination:

```bash
test -f "$CM_SOURCE_DIR/WinCM.exe" || {
  printf 'Source installation not found: %s\n' "$CM_SOURCE_DIR" >&2
  exit 1
}

test ! -e "$CM_DEST_DIR" || {
  printf 'Destination already exists: %s\n' "$CM_DEST_DIR" >&2
  exit 1
}
```

Copy the entire directory, not only `WinCM.exe`:

```bash
mkdir -p "$CM_DEST_PARENT"
cp -a "$CM_SOURCE_DIR" "$CM_DEST_PARENT/"
```

Verify the relocated executable:

```bash
ls -lh "$CM_EXE"
```

Launch the relocated copy directly:

```bash
wine "$CM_EXE"
```

Enter the valid licence in CrystalMaker's normal activation interface. Do not
put licence codes in shell history, scripts, screenshots, or this document.

### 14.4 Why changing MSI properties after installation was insufficient

The inspected build checked the executable location before consulting an
`AllUsers` registry value. If `WinCM.exe` was inside `Program Files` or
`Program Files (x86)`, it was immediately treated as per-machine.

Consequently, modifying `MSIINSTALLPERUSER` after installation did not change
the running executable's location and did not solve the tested licence message.

> [!WARNING]
> This is not a licence bypass. It makes the installation layout match the
> single-user requirement enforced by the application while still requiring a
> valid licence and normal activation. If your organisation's licence terms or
> vendor instructions differ, follow them or contact CrystalMaker support.

## 15. Install and register Segoe UI fonts

### 15.1 Obtain the font files legally

CrystalMaker uses Segoe UI in several WPF components. Copy the fonts from a
legitimate Windows installation or licensed Windows image.

Minimum tested/recommended set:

```text
segoeui.ttf     Segoe UI Regular
segoeuib.ttf    Segoe UI Bold
segoeuii.ttf    Segoe UI Italic
segoeuiz.ttf    Segoe UI Bold Italic
seguisym.ttf    Segoe UI Symbol
seguiemj.ttf    Segoe UI Emoji (optional)
```

Set the directory containing the legally obtained files:

```bash
export WINDOWS_FONT_SOURCE="/path/to/windows-fonts"
export WINE_FONT_DIR="$WINEPREFIX/drive_c/windows/Fonts"
```

Check the minimum set before copying:

```bash
for font in segoeui.ttf segoeuib.ttf segoeuii.ttf segoeuiz.ttf seguisym.ttf; do
  test -f "$WINDOWS_FONT_SOURCE/$font" || {
    printf 'Missing source font: %s\n' "$font" >&2
    exit 1
  }
done
```

Copy the fonts:

```bash
install -m 0644 "$WINDOWS_FONT_SOURCE/segoeui.ttf"  "$WINE_FONT_DIR/segoeui.ttf"
install -m 0644 "$WINDOWS_FONT_SOURCE/segoeuib.ttf" "$WINE_FONT_DIR/segoeuib.ttf"
install -m 0644 "$WINDOWS_FONT_SOURCE/segoeuii.ttf" "$WINE_FONT_DIR/segoeuii.ttf"
install -m 0644 "$WINDOWS_FONT_SOURCE/segoeuiz.ttf" "$WINE_FONT_DIR/segoeuiz.ttf"
install -m 0644 "$WINDOWS_FONT_SOURCE/seguisym.ttf" "$WINE_FONT_DIR/seguisym.ttf"
```

Copy the optional Emoji font if available:

```bash
if test -f "$WINDOWS_FONT_SOURCE/seguiemj.ttf"; then
  install -m 0644 \
    "$WINDOWS_FONT_SOURCE/seguiemj.ttf" \
    "$WINE_FONT_DIR/seguiemj.ttf"
fi
```

### 15.2 Register Segoe UI in Wine

Copying files into `C:\Windows\Fonts` did not automatically create every
required Wine registry entry in the tested prefix.

Register the core family:

```bash
wine reg add \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  /v 'Segoe UI (TrueType)' \
  /t REG_SZ \
  /d 'segoeui.ttf' \
  /f

wine reg add \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  /v 'Segoe UI Bold (TrueType)' \
  /t REG_SZ \
  /d 'segoeuib.ttf' \
  /f

wine reg add \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  /v 'Segoe UI Italic (TrueType)' \
  /t REG_SZ \
  /d 'segoeuii.ttf' \
  /f

wine reg add \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  /v 'Segoe UI Bold Italic (TrueType)' \
  /t REG_SZ \
  /d 'segoeuiz.ttf' \
  /f

wine reg add \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  /v 'Segoe UI Symbol (TrueType)' \
  /t REG_SZ \
  /d 'seguisym.ttf' \
  /f
```

Register Segoe UI Emoji if copied:

```bash
if test -f "$WINE_FONT_DIR/seguiemj.ttf"; then
  wine reg add \
    'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
    /v 'Segoe UI Emoji (TrueType)' \
    /t REG_SZ \
    /d 'seguiemj.ttf' \
    /f
fi
```

### 15.3 Remove an incorrect substitution

Inspect the font substitution key:

```bash
wine reg query \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes'
```

Query Segoe UI specifically:

```bash
wine reg query \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes' \
  /v 'Segoe UI'
```

If it explicitly substitutes Segoe UI with Tahoma or Arial after the genuine
Segoe fonts have been installed, remove that value:

```bash
wine reg delete \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes' \
  /v 'Segoe UI' \
  /f
```

Do not delete unrelated font substitutions.

### 15.4 Refresh and verify fonts

```bash
wineserver -k
fc-cache -f "$WINE_FONT_DIR"
wineboot -u
wineserver -k
```

Verify the files:

```bash
find "$WINE_FONT_DIR" \
  -maxdepth 1 -type f \
  \( -iname 'segoe*' -o -iname 'segui*' \) \
  -printf '%f\n' \
  | sort
```

Verify the Wine registry:

```bash
wine reg query \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  | grep -i 'Segoe UI'
```

Open Wine's font control panel if visual confirmation is required:

```bash
wine control fonts
```

`fc-match 'Segoe UI'` checks Linux Fontconfig and may still name a Linux fallback
font. For CrystalMaker, the Wine font directory and Wine registry are the more
important checks.

## 16. Create a native GNOME launcher

Wine's generated file association may still point to the old `Program Files`
copy. Launch the relocated `WinCM.exe` directly through a wrapper script.

### 16.1 Create the wrapper directory

```bash
mkdir -p "$HOME/.local/bin"
```

Create this file in a text editor:

```text
~/.local/bin/crystalmaker
```

Example using GNOME Text Editor:

```bash
gnome-text-editor "$HOME/.local/bin/crystalmaker"
```

Use the following content, replacing `YOUR_WINE_USERNAME` with the value returned
by `wine cmd /c 'echo %USERNAME%'`:

```bash
#!/usr/bin/env bash
set -euo pipefail

export WINEPREFIX="$HOME/.wine-crystalmaker"

WINE_BIN="${WINE_BIN:-$(command -v wine)}"
CM_WINE_USER="YOUR_WINE_USERNAME"
CM_EXE="$WINEPREFIX/drive_c/users/$CM_WINE_USER/AppData/Local/Programs/CrystalMaker Software/CrystalMaker/WinCM.exe"

if [[ ! -f "$CM_EXE" ]]; then
  printf 'CrystalMaker executable not found: %s\n' "$CM_EXE" >&2
  exit 1
fi

exec "$WINE_BIN" "$CM_EXE" "$@"
```

Make it executable:

```bash
chmod +x "$HOME/.local/bin/crystalmaker"
```

Test it from a terminal:

```bash
"$HOME/.local/bin/crystalmaker"
```

The wrapper intentionally does not run `wineserver -k` on every start. Killing
Wine automatically can terminate other Windows applications using Wine.

### 16.2 Create the desktop entry

Create the applications directory:

```bash
mkdir -p "$HOME/.local/share/applications"
```

Create:

```text
~/.local/share/applications/crystalmaker.desktop
```

Use this content:

```ini
[Desktop Entry]
Version=1.0
Type=Application
Name=CrystalMaker 11
Comment=Crystal and molecular structures
Exec=/home/YOUR_LINUX_USERNAME/.local/bin/crystalmaker
Icon=85C7_WinCM.0
Terminal=false
StartupNotify=true
Categories=Science;Education;
```

Replace `YOUR_LINUX_USERNAME` with the Linux account name shown by:

```bash
id -un
```

The `Exec=` value must remain on one physical line. Desktop entries do not use
shell continuation backslashes in the `Exec` key.

If `85C7_WinCM.0` is unavailable, use a full path to a legally obtained/extracted
CrystalMaker icon or use a generic icon such as:

```ini
Icon=applications-science
```

Validate and update the desktop database:

```bash
desktop-file-validate \
  "$HOME/.local/share/applications/crystalmaker.desktop"

update-desktop-database \
  "$HOME/.local/share/applications"
```

The application should then appear in GNOME's application overview. Logging out
is normally unnecessary after updating the desktop database.

## 17. Verify the final installation

### 17.1 Configuration checks

```bash
wine --version
```

```bash
wine reg query \
  'HKCU\Software\Wine\X11 Driver' \
  /v UseEGL
```

```bash
wine reg query \
  'HKCU\Software\Microsoft\Avalon.Graphics' \
  /v DisableHWAcceleration
```

```bash
test -f "$CM_EXE" && printf 'Per-user WinCM.exe is present\n'
```

```bash
wine reg query \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  | grep -i 'Segoe UI'
```

### 17.2 Functional checklist

- [ ] CrystalMaker starts from the custom GNOME launcher.
- [ ] The launcher opens the copy under `AppData\Local\Programs`.
- [ ] The valid group licence activates normally.
- [ ] The main WPF interface is fully drawn.
- [ ] Highlights/Gallery is visible.
- [ ] Carbon - Diamond opens without a crash.
- [ ] The crystal can be rotated, zoomed, and panned.
- [ ] CrystalMaker identifies the NVIDIA renderer.
- [ ] CrystalMaker reports a usable OpenGL mode.
- [ ] Segoe UI punctuation and symbols render without square boxes.
- [ ] Five cold-start tests succeed.

### 17.3 Cold-start test

Run this manually five times:

```bash
wineserver -k
sleep 2
"$HOME/.local/bin/crystalmaker"
```

Each time:

1. open Diamond;
2. rotate the structure;
3. close the structure;
4. close CrystalMaker normally.

Five successful runs provide stronger evidence that the clean-prefix setup is
repeatable and not relying on an accidental stale process state.

## 18. Back up the working prefix

Close Wine before copying or archiving the prefix:

```bash
wineserver -k
```

Create a direct copy:

```bash
cp -a \
  "$HOME/.wine-crystalmaker" \
  "$HOME/.wine-crystalmaker-working"
```

Or create a compressed snapshot:

```bash
tar --zstd \
  -cf "$HOME/crystalmaker-wine11.13-working.tar.zst" \
  -C "$HOME" \
  .wine-crystalmaker
```

The prefix may contain licence state and user data. Store backups securely and
do not publish them.

For future experiments, copy the working prefix and change the copy:

```bash
cp -a \
  "$HOME/.wine-crystalmaker-working" \
  "$HOME/.wine-crystalmaker-test"
```

## 19. Troubleshooting

### 19.1 `wglCreateContext` or pixel-format failure

Typical log messages include:

```text
wglCreateContext failed
Failed to make current
Failed to set pixel format
GL version 0.0
```

Reapply and verify the GLX setting:

```bash
wine reg add \
  'HKCU\Software\Wine\X11 Driver' \
  /v UseEGL \
  /t REG_SZ \
  /d N \
  /f

wineserver -k
```

Also verify the native Linux OpenGL renderer with `glxinfo -B`.

### 19.2 CrystalMaker starts but freezes or crashes when opening a crystal

Verify WPF hardware acceleration is disabled:

```bash
wine reg add \
  'HKCU\Software\Microsoft\Avalon.Graphics' \
  /v DisableHWAcceleration \
  /t REG_DWORD \
  /d 1 \
  /f

wineserver -k
```

Confirm WineHQ Development 11.13 is actually being used. The earlier tested
Wine 11.0 builds remained unstable after OpenGL context creation was repaired.

### 19.3 Black interface, partial redraw, or content appears after scrolling

This was the observed DXVK failure mode with CrystalMaker's WPF interface.

Use a clean prefix without DXVK or VKD3D. If testing an old disposable prefix,
restore WineD3D and remove stale native overrides as described in Section 12.

### 19.4 Gallery works on one monitor but a crystal fails on another

First confirm:

- WineHQ Development 11.13 is in use;
- `UseEGL=N` is present;
- WPF hardware acceleration is disabled;
- DXVK/VKD3D are absent;
- both displays use the intended scale and refresh configuration.

Collect the monitor state:

```bash
gsettings get org.gnome.desktop.interface scaling-factor
gsettings get org.gnome.mutter experimental-features
```

Test the clean prefix before adding more registry or GPU-offload variables. The
final stable setup did not require rewriting values before every launch.

### 19.5 CrystalMaker immediately exits

Check the Windows Desktop Runtime:

```bash
find \
  "$WINEPREFIX/drive_c/Program Files/dotnet/shared/Microsoft.WindowsDesktop.App" \
  -maxdepth 1 -mindepth 1 -type d -printf '%f\n'
```

Confirm that the x64 Desktop Runtime 8 installer completed successfully.

Generate a warning log:

```bash
wineserver -k

WINEDEBUG=warn+all \
"$HOME/.local/bin/crystalmaker" \
  &> "$HOME/crystalmaker-warn.log"
```

### 19.6 MSI installation fails

Confirm the MSI exists and is readable:

```bash
ls -lh "$CM_INSTALLER"
file "$CM_INSTALLER"
```

Create a verbose MSI log:

```bash
wine msiexec /i \
  "$CM_INSTALLER" \
  '/L*V' \
  "$HOME/WinCM-install.log"
```

Inspect the end of the log:

```bash
tail -200 "$HOME/WinCM-install.log"
```

Use the normal `/i` installation path. Do not substitute an administrative `/a`
extraction unless specifically diagnosing MSI contents.

### 19.7 Licence says it is valid only for a single-user installation

Check the executable actually launched:

```bash
printf '%s\n' "$CM_EXE"
```

It must be under:

```text
C:\Users\<wine-user>\AppData\Local\Programs\...
```

Do not launch the old Program Files copy through Wine's generated ProgID file
association. Use the wrapper script and direct executable path.

If the message remains, confirm the licence type and installation terms with
CrystalMaker support rather than attempting to alter activation data.

### 19.8 Square boxes replace punctuation or symbols

Confirm that `segoeui.ttf` and `seguisym.ttf` exist and are registered:

```bash
find "$WINEPREFIX/drive_c/windows/Fonts" \
  -maxdepth 1 -type f \
  \( -iname 'segoe*' -o -iname 'segui*' \)
```

```bash
wine reg query \
  'HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts' \
  | grep -i 'Segoe UI'
```

Then restart Wine. If ordinary ASCII characters still render as boxes, the
problem may instead be CrystalMaker's log control, rich-text conversion, or the
Wine WPF text path.

### 19.9 Desktop launcher fails validation or disappears

Validate it:

```bash
desktop-file-validate \
  "$HOME/.local/share/applications/crystalmaker.desktop"
```

Common error: splitting `Exec=` over several lines with `\`. The `Exec=` command
must be one line. Keeping complex path handling in the wrapper script avoids
Desktop Entry escaping problems.

Refresh GNOME's application database:

```bash
update-desktop-database "$HOME/.local/share/applications"
```

### 19.10 Collect a focused crash log

Start with warnings:

```bash
wineserver -k

WINEDEBUG=warn+all \
"$HOME/.local/bin/crystalmaker" \
  &> "$HOME/crystalmaker-warn.log"
```

For OpenGL failures:

```bash
wineserver -k

WINEDEBUG=+opengl \
"$HOME/.local/bin/crystalmaker" \
  &> "$HOME/crystalmaker-opengl.log"
```

Extract useful lines:

```bash
grep -nEi \
  'wglCreateContext|wglMakeCurrent|GL version|OpenGL context|No active WGL|failed|Unhandled|Process terminated|FailFast|e0434352|page fault' \
  "$HOME/crystalmaker-opengl.log" \
  | tail -100
```

Do not publish logs until they have been checked for usernames, file paths,
licence data, and other private information.

## 20. Reverse-engineering notes

These notes explain the compatibility finding that guided the per-user layout.
They are not instructions for bypassing activation.

The inspected assemblies were:

```text
WinCM.dll
CMUtils.dll
```

The relevant classes/methods included:

```text
RegistryHelper.IsPerMachineInstallation()
RegistryHelper.IsInstalledInProgramFiles()
CMLicense.Authenticate(...)
CMUniLicence
```

The observed logic can be summarised as:

```csharp
bool IsPerMachineInstallation()
{
    if (IsInstalledInProgramFiles())
        return true;

    // Only checked if the executable is not under Program Files.
    return ReadAllUsersRegistryValue() == 1;
}
```

The Program Files test was equivalent in intent to:

```csharp
bool IsInstalledInProgramFiles()
{
    string location = Assembly.GetEntryAssembly().Location;

    return location.StartsWith(
               Environment.GetFolderPath(
                   Environment.SpecialFolder.ProgramFiles))
        || location.StartsWith(
               Environment.GetFolderPath(
                   Environment.SpecialFolder.ProgramFilesX86));
}
```

Only when the executable was outside Program Files did the application inspect
an `AllUsers` value under a CrystalMaker registry key.

The tested licence classes requiring a non-per-machine layout included:

```text
EducationPersonal
EducationGroup
StandardPersonal
StandardGroup
Student
```

Therefore, for the inspected version:

```text
Group licence != all-users/per-machine installation
```

Moving the complete application directory to the Wine user's
`AppData\Local\Programs` directory caused the executable-location test to return
false and allowed the valid group licence to activate through the normal UI.

This implementation is version-specific. Re-check vendor documentation and the
current executable if a future CrystalMaker update changes the behaviour.

## 21. Registry reference

### OpenGL backend

```text
Key:   HKCU\Software\Wine\X11 Driver
Value: UseEGL
Type:  REG_SZ
Data:  N
```

Command:

```bash
wine reg add \
  'HKCU\Software\Wine\X11 Driver' \
  /v UseEGL /t REG_SZ /d N /f
```

### WPF interface composition

```text
Key:   HKCU\Software\Microsoft\Avalon.Graphics
Value: DisableHWAcceleration
Type:  REG_DWORD
Data:  1
```

Command:

```bash
wine reg add \
  'HKCU\Software\Microsoft\Avalon.Graphics' \
  /v DisableHWAcceleration /t REG_DWORD /d 1 /f
```

### Wine font registration

```text
Key: HKLM\Software\Microsoft\Windows NT\CurrentVersion\Fonts
```

Core values:

```text
Segoe UI (TrueType)             -> segoeui.ttf
Segoe UI Bold (TrueType)        -> segoeuib.ttf
Segoe UI Italic (TrueType)      -> segoeuii.ttf
Segoe UI Bold Italic (TrueType) -> segoeuiz.ttf
Segoe UI Symbol (TrueType)      -> seguisym.ttf
Segoe UI Emoji (TrueType)       -> seguiemj.ttf
```

### CrystalMaker installation classification

The inspected build consulted an `AllUsers` value similar to:

```text
HKLM\Software\CrystalMaker Software\CrystalMaker11\AllUsers
```

However, an executable located under Program Files was classified as per-machine
before that fallback check.

## 22. Maintenance and upgrades

### Update .NET 8 servicing releases

Install a newer supported .NET Desktop Runtime 8 x64 installer into the same
prefix, then verify the runtime directory and retest CrystalMaker.

### Test a newer Wine build

Do not upgrade the only working prefix first.

1. Close Wine.
2. Copy the working prefix or create a new clean prefix.
3. Select the newer Wine binary explicitly.
4. Run `wineboot -u` against the test prefix.
5. Repeat the five-run functional checklist.
6. Keep the original prefix until the new build is proven stable.

Example test copy:

```bash
wineserver -k

cp -a \
  "$HOME/.wine-crystalmaker-working" \
  "$HOME/.wine-crystalmaker-wine-upgrade-test"

export WINEPREFIX="$HOME/.wine-crystalmaker-wine-upgrade-test"
```

### Update CrystalMaker

Use a separate test prefix for major application updates. The licence-location
logic, .NET requirement, fonts, file associations, and executable layout may
change between CrystalMaker versions.

### Keep a configuration record

Record at least:

```bash
wine --version
rpm -q winehq-devel wine-devel
uname -r
nvidia-smi
```

Also record:

- Fedora version;
- GNOME session type;
- CrystalMaker version;
- .NET Desktop Runtime patch version;
- the two registry values from Section 21;
- whether Segoe UI was installed and registered;
- the per-user executable path.

## 23. References

- [WineHQ Fedora 44 repository file](https://dl.winehq.org/wine-builds/fedora/44/winehq.repo)
- [Microsoft .NET 8 downloads](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
- [Winetricks project](https://github.com/Winetricks/winetricks)
- [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry-spec/latest/)

## Final known-good configuration

```text
Fedora 44 Workstation
GNOME Wayland
NVIDIA RTX 3080 with native Linux driver
WineHQ Development 11.13
Clean win64 Wine prefix
Windows 10 mode
UseEGL=N
DisableHWAcceleration=1
Microsoft .NET Desktop Runtime 8 x64
corefonts + tahoma
WineD3D (no DXVK, no VKD3D)
CrystalMaker executable under AppData\Local\Programs
Valid group licence activated normally
Segoe UI + Segoe UI Symbol registered
Custom wrapper and GNOME desktop entry
```

With this configuration, the tested system started CrystalMaker reliably,
rendered crystals through the NVIDIA OpenGL stack, activated the valid group
licence from a per-user installation path, displayed the intended fonts, and
launched cleanly from GNOME.
