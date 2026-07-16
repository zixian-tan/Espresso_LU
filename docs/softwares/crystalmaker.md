# Installing CrystalMaker 11 on Fedora Linux using Wine

This guide describes a verified installation procedure for **CrystalMaker 11** on **Fedora 44** using **WineHQ Development**.

The following configuration has been tested successfully:

| Component | Version |
|-----------|---------|
| Fedora | 44 |
| GNOME | Wayland |
| NVIDIA Driver | 595.xx |
| Wine | WineHQ Development 11.13 |
| CrystalMaker | 11.0 |

---

# 1. Install WineHQ Development

Follow the official WineHQ installation guide for Fedora.

Verify the installation:

```bash
wine --version
```

Expected output:

```text
wine-11.13
```

---

# 2. Create a new Wine prefix

```bash
export WINEPREFIX="$HOME/.wine-crystalmaker"
export WINEARCH=win64

wineboot -i
```

Open Wine configuration:

```bash
winecfg
```

Select:

```
Windows 10
```

---

# 3. Configure Wine

CrystalMaker requires two registry modifications.

## 3.1 Disable EGL

Wine's Wayland EGL backend may cause OpenGL context creation failures.

```bash
wine reg add \
'HKCU\Software\Wine\X11 Driver' \
/v UseEGL \
/t REG_SZ \
/d N \
/f
```

Verify:

```bash
wine reg query \
'HKCU\Software\Wine\X11 Driver' \
/v UseEGL
```

Expected:

```
UseEGL    REG_SZ    N
```

---

## 3.2 Disable WPF hardware acceleration

CrystalMaker uses a WPF user interface.

Wine's Direct3D implementation may cause the application to freeze after opening a crystal.

Disable WPF hardware rendering:

```bash
wine reg add \
'HKCU\Software\Microsoft\Avalon.Graphics' \
/v DisableHWAcceleration \
/t REG_DWORD \
/d 1 \
/f
```

This only affects the WPF interface.

The OpenGL crystal renderer continues to use the GPU.

---

# 4. Install required fonts

Install winetricks dependencies:

```bash
sudo dnf install cabextract
```

Install common Microsoft fonts:

```bash
winetricks -q corefonts tahoma
```

---

# 5. Install Microsoft .NET Desktop Runtime

CrystalMaker 11 requires the Microsoft .NET Desktop Runtime.

Download the Windows x64 installer from Microsoft and install:

```bash
wine windowsdesktop-runtime-8.x.x-win-x64.exe
```

---

# 6. Install CrystalMaker

```bash
wine msiexec /i WinCM.msi
```

After installation, launch:

```bash
wine \
"$WINEPREFIX/drive_c/Program Files/CrystalMaker Software/CrystalMaker/WinCM.exe"
```

---

# Final working configuration

The following configuration has been verified to work:

| Setting | Value |
|----------|------|
| Wine | WineHQ Development 11.13 |
| Windows version | Windows 10 |
| UseEGL | N |
| DisableHWAcceleration | 1 |
| Fonts | corefonts, tahoma |
| Runtime | Microsoft .NET Desktop Runtime 8 |
| Graphics backend | WineD3D |

---

# Notes

- DXVK is **not recommended**.
- VKD3D is **not required**.
- Bottles and Proton were not used in the final configuration.
- The registry modifications only need to be applied **once** when creating the Wine prefix.


# Troubleshooting

## OpenGL context creation failed

Symptoms:

- `wglCreateContext` failed
- `Failed to make current`
- `Failed to set pixel format`

Solution:

```bash
wine reg add \
'HKCU\Software\Wine\X11 Driver' \
/v UseEGL \
/t REG_SZ \
/d N \
/f
```

---

## Application freezes after opening a crystal

Symptoms:

- CrystalMaker starts normally.
- Opening a crystal causes the application to freeze.

Solution:

```bash
wine reg add \
'HKCU\Software\Microsoft\Avalon.Graphics' \
/v DisableHWAcceleration \
/t REG_DWORD \
/d 1 \
/f
```