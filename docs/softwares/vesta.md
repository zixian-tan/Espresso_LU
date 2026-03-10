---
sidebar_position: 1
---
# VESTA Installation Guide

> For Ubuntu 24.04-25.10 and fedora 43 workstation

VESTA is a visualization program for crystal structures and electronic densities.  
These steps describe installation on Ubuntu 25.01 using the official precompiled Linux package (`VESTA-gtk3-x86_64.tar.bz2`).

---

## 1️⃣ Download and Extract

```bash
cd ~/Downloads
tar -xvf VESTA-gtk3-x86_64.tar.bz2
sudo mv VESTA-gtk3-x86_64 /opt/vesta
```

---

## 2️⃣ Create an Executable Link

```bash
sudo ln -s /opt/vesta/VESTA /usr/local/bin/vesta
```

---

## 3️⃣ Install Required Dependencies

For Ubuntu :

```bash
sudo apt update
sudo apt install libglu1-mesa libx11-6 libxext6 libxt6 libxrender1 libsm6 libwebkit2gtk-4.1-0 libjavascriptcoregtk-4.1-0
sudo ln -s /usr/lib/x86_64-linux-gnu/libwebkit2gtk-4.1.so.0 /usr/lib/x86_64-linux-gnu/libwebkit2gtk-4.0.so.37
sudo ln -s /usr/lib/x86_64-linux-gnu/libjavascriptcoregtk-4.1.so.0 /usr/lib/x86_64-linux-gnu/libjavascriptcoregtk-4.0.so.18
```

For fedora:

```bash
sudo dnf update
sudo dnf install -y mesa-libGLU libX11 libXext libXt libXrender libSM webkit2gtk4.1 javascriptcoregtk4.1
sudo ln -s /usr/lib64/libwebkit2gtk-4.1.so.0 /usr/lib64/libwebkit2gtk-4.0.so.37
sudo ln -s /usr/lib64/libjavascriptcoregtk-4.1.so.0 /usr/lib64/libjavascriptcoregtk-4.0.so.18
```

---

## 4️⃣ Create a Desktop Launcher

```bash
sudo vi /usr/share/applications/vesta.desktop
```

```ini
[Desktop Entry]
Name=VESTA
Comment=Visualization for Electronic and Structural Analysis
Exec=/opt/vesta/VESTA
Icon=/opt/vesta/img/logo@2x.png
Terminal=false
Type=Application
Categories=Science;Education;Chemistry;Physics;
StartupWMClass=Vesta-gui
```

```bash
sudo chmod +x /usr/share/applications/vesta.desktop
sudo update-desktop-database
```

---

## 5️⃣ Verify Installation

```bash
vesta
```

Or you can directly open VESTA by click the icon in Application list.
