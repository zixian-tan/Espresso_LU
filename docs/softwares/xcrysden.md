---
sidebar_position: 2
---
# XCrySDen Installation and Compatibility Guide

> Ubuntu 24.04-25.10 and fedora 43 workstation

XCrySDen is a crystal and molecular structure visualization tool based on Tcl/Tk.  
This guide is for version `1.6.2` (latest stable).

---

## 1️⃣ Download the Precompiled Package

```bash
xcrysden-1.6.2-linux_x86_64-shared.tar.gz
```

---

## 2️⃣ Extract and Move

```bash
cd ~/Downloads
tar -xvf xcrysden-1.6.2-linux_x86_64-shared.tar.gz
sudo mv xcrysden-1.6.2-bin-shared /opt/xcrysden
```

---

## 3️⃣ Install Dependencies

For Ubuntu:

```bash
sudo apt update
sudo apt install tk libglu1-mesa libtogl2 libxmu6 imagemagick openbabel libgfortran5 libfftw3-dev
```

For Fedora, you should download Togl2.0-8.4-Linux64 first

```bash
tar -xzf Togl2.0-8.4-Linux64.tar.gz
sudo mkdir -p /opt/togl2
sudo cp -a Togl2.0-8.4-Linux/* /opt/togl2/
cd /opt/togl2/lib/Togl2.0
sudo ln -sf libTogl2.0.so libTogl.so.2
echo "/opt/togl2/lib/Togl2.0" | sudo tee /etc/ld.so.conf.d/togl2.conf 
sudo ldconfig
```

---

<!-- ## 4️⃣ Fix Compatibility (Togl & Wayland)

Find and link Togl library:

```bash
dpkg -L libtogl2 | grep -i '\.so'
sudo ln -s /usr/lib/x86_64-linux-gnu/Togl2.0/libTogl2.0.so /usr/lib/x86_64-linux-gnu/libTogl.so.2
```

Configure the following environment variables:

```bash
vi ~/.bashrc
```

Add this command at the end of file.

```ini
alias xcrysden='env GDK_BACKEND=x11 /opt/xcrysden/xcrysden'
```

Save it and do:

```s
source ~/.bashrc
```

--- -->

## 4️⃣ Disable Incompatible Togl Options

```bash
mkdir -p ~/.xcrysden
cp /opt/xcrysden/Tcl/custom-definitions ~/.xcrysden/
vi ~/.xcrysden/custom-definitions
```

Add the following lines:

```vi
# Safe Togl options for modern systems (Wayland/XWayland)
set toglOpt(rgba)   true
set toglOpt(double) true
set toglOpt(depth)  true

set toglOpt(accum)   false
set toglOpt(stencil) false
set toglOpt(stereo)  false
set toglOpt(overlay) false
```

## 5️⃣ Create a Desktop Shortcut

```bash
sudo vi /usr/share/applications/xcrysden.desktop
```

```ini
[Desktop Entry]
Name=XCrySDen
Comment=Crystalline and Molecular Structure Visualization
Exec=/opt/xcrysden/xcrysden
Icon=/opt/xcrysden/images/xcrysden.png
Terminal=false
Type=Application
Categories=Science;Education;Chemistry;Physics;
StartupWMClass=Xcinit.tcl
```

```bash
sudo chmod +x /usr/share/applications/xcrysden.desktop
sudo update-desktop-database
```

---

## 6️⃣ Verify Installation

```bash
xcrysden
```

## 7️⃣ Extra Step (For Ubuntu)

The imagemask icon name will be shown after installation, but the name is wrong.

![icon_err](../../img/icon_err.jpeg)

```bash
sudo vi /usr/share/applications/display-im6.q16.desktop
```

```vi
Name = ImageMagick Viewer
```

```bash
sudo update-desktop-database
```
