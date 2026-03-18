---
sidebar_position: 4
---
# BURAI

> Installing BURAI and Fixing JavaFX Compatibility Issues on Ubuntu (22.04 / 24.04)

BURAI is a JavaFX-based graphical interface for Quantum ESPRESSO.
However, modern Ubuntu systems (20.04 and later) ship OpenJDK without JavaFX, which leads to launch failures such as:

```bash
Error: Could not find or load main class burai.app.QEFXMain
Caused by: java.lang.NoClassDefFoundError: javafx/application/Application
```

This version is tailored to the following environment:

- BURAI installation path: ``/usr/share/applications/BURAI/``
- JavaFX installed via: ``/usr/share/openjfx/lib/*.jar``
- All BURAI ``.jar`` files located in: ``/usr/share/applications/BURAI/bin/``
- Default system Java: ``OpenJDK 21``

## 1️⃣ Install BURAI

Download BURAI (e.g BURAI 1.3)

```bash
wget https://github.com/QEFix/BURAI/releases/download/v1.3/BURAI-1.3-linux.tar.gz
```

Extract to installation directory:

```bash
sudo mkdir -p /usr/share/applications/BURAI
sudo tar -xzf BURAI-1.3-linux.tar.gz -C /usr/share/applications/BURAI --strip-components=1
```

Check the directory structure, the expected result is: 

```bash
BURAI/
 ├── bin/
 │    ├── BURAI.jar
 │    └── other .jar files
 ├── lib/
 ├── icons/
 ├── burai.desktop
 └── ...
```

Your installation places all ``.jar`` files inside ``BURAI/bin``, which requires a custom classpath (addressed in Section 4).

## 2️⃣ Install JavaFX
