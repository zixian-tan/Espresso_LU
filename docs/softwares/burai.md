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

OpenJDK on Ubuntu does not bundle JavaFX. Install it separately:

```bash
sudo apt update
sudo apt install openjfx
```

Verify installation:

```bash
dpkg -L openjfx | grep .jar
```

Typical output:

```bash
/usr/share/openjfx/lib/javafx.base.jar
/usr/share/openjfx/lib/javafx.controls.jar
/usr/share/openjfx/lib/javafx.fxml.jar
...
```

JavaFX is now available at:

```bash
/usr/share/openjfx/lib
```

## 3️⃣ Rewriting the BURAI Launch Script (Critical Fix)

Edit the launch script:

```bash
sudo vi /usr/share/applications/BURAI/bin/burai.sh
```

Replace the entire file with the following fully working and environment-specific script:

```bash
#!/bin/sh

# === Path Configuration ===
# BURAI root directory (this script is inside BURAI/bin)
BURAI_HOME="$(cd "$(dirname "$0")/.." && pwd)"

# Java command (system default OpenJDK 21)
JAVA_CMD=/usr/bin/java

# JavaFX module path (provided by the openjfx package)
JFX_PATH=/usr/share/openjfx/lib

# === Launch BURAI ===
exec "$JAVA_CMD" \
  --module-path "$JFX_PATH" \
  --add-modules=javafx.controls,javafx.fxml,javafx.graphics,javafx.web,javafx.media,javafx.swing \
  -cp "$BURAI_HOME/bin/*:$BURAI_HOME/lib/*" \
  burai.app.QEFXMain "$@"
```

Save and exit.
Make it executable:

```bash
sudo chmod +x /usr/share/applications/BURAI/bin/burai.sh
```

## 4️⃣ Running BURAI
Launch BURAI:

```bash
cd /usr/share/applications/BURAI/bin
./burai.sh
```

If JavaFX is correctly loaded, BURAI will start normally with full GUI functionality.

## 5️⃣ Create Desktop file

The same as below

---

# Installing BURAI on Fedora 44 (GNOME / Wayland)

This guide explains how to install and run BURAI on Fedora 44, including:

- Java compatibility fixes
- JavaFX setup
- Wayland/X11 compatibility
- Desktop launcher creation

BURAI is an old JavaFX application and does not work correctly with Fedora 44 default Java 25/26 runtime out of the box.

---

# 1. Extract BURAI

Download the Linux version of BURAI and extract it.

Example:

```bash
cd ~/Downloads
tar -xvf BURAI*.tar*
```

Move it to a system-wide location:

```bash
sudo mv BURAI /usr/share/applications/BURAI
```

The final structure should look similar to:

```text
/usr/share/applications/BURAI/
├── bin
│   ├── burai.jar
│   ├── burai.png
│   └── burai.sh
├── examples
├── exec.LINUX
├── pseudopot
└── ...
```

---

# 2. Install JavaFX

Fedora 44 already includes OpenJDK 25 by default, but BURAI is incompatible with it.

Install Fedora JavaFX package first:

```bash
sudo dnf install openjfx
```

Verify installation:

```bash
rpm -ql openjfx | grep 'javafx.*jar'
```

Expected output:

```text
/usr/lib/jvm/openjfx/javafx.base.jar
/usr/lib/jvm/openjfx/javafx.controls.jar
/usr/lib/jvm/openjfx/javafx.fxml.jar
/usr/lib/jvm/openjfx/javafx.graphics.jar
/usr/lib/jvm/openjfx/javafx.web.jar
...
```

---

# 3. Install Java 17 (Temurin)

Fedora 44 repositories no longer provide OpenJDK 17 directly.

Install Eclipse Temurin 17 instead.

## Add Adoptium repository

```bash
sudo rpm --import https://packages.adoptium.net/artifactory/api/gpg/key/public
```

```bash
sudo tee /etc/yum.repos.d/adoptium.repo > /dev/null << 'EOF'
[Adoptium]
name=Adoptium
baseurl=https://packages.adoptium.net/artifactory/rpm/fedora/$releasever/$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.adoptium.net/artifactory/api/gpg/key/public
EOF
```

## Install Temurin 17

```bash
sudo dnf install temurin-17-jdk
```

Verify installation:

```bash
/usr/lib/jvm/temurin-17-jdk/bin/java -version
```

Expected output:

```text
openjdk version "17.x.x"
```

---

# 4. Install JavaFX 17 SDK

Fedora's system JavaFX is compiled for Java 25+, which is incompatible with Java 17.

Therefore a matching JavaFX 17 SDK must be installed manually.

## Download JavaFX 17

```bash
cd ~/Downloads
wget https://download2.gluonhq.com/openjfx/17.0.16/openjfx-17.0.16_linux-x64_bin-sdk.zip
```

## Extract

```bash
unzip openjfx-17.0.16_linux-x64_bin-sdk.zip
```

## Move to system location

```bash
sudo mv javafx-sdk-17.0.16 /opt/
```

Final location:

```text
/opt/javafx-sdk-17.0.16
```

---

# 5. Configure BURAI Launcher Script

Edit launcher:

```bash
sudo vi /usr/share/applications/BURAI/bin/burai.sh
```

Replace contents with:

```bash
#!/usr/bin/env bash
cd "$(dirname "$0")"

JAVA_HOME=/usr/lib/jvm/temurin-17-jdk
JAVAFX_HOME=/opt/javafx-sdk-17.0.16

GDK_BACKEND=x11 \
"$JAVA_HOME/bin/java" \
  --module-path "$JAVAFX_HOME/lib" \
  --add-modules javafx.controls,javafx.fxml,javafx.graphics,javafx.web,javafx.swing \
  -jar burai.jar
```

Make executable:

```bash
sudo chmod +x /usr/share/applications/BURAI/bin/burai.sh
```

---

# 6. Launch BURAI

Run:

```bash
/usr/share/applications/BURAI/bin/burai.sh
```

BURAI should now open correctly.

---

# 7. Create Desktop Launcher

Create desktop entry:

```bash
sudo vi /usr/share/applications/burai.desktop
```

Insert:

```ini
[Desktop Entry]
Version=1.0
Type=Application
Name=BURAI
Comment=Quantum ESPRESSO GUI
Exec=/usr/share/applications/BURAI/bin/burai.sh
Icon=/usr/share/applications/BURAI/bin/burai.png
Terminal=false
Categories=Science;Education;
StartupWMClass=burai.app.QEFXMain
```

Make executable:

```bash
sudo chmod +x /usr/share/applications/burai.desktop
```

Refresh desktop database:

```bash
update-desktop-database ~/.local/share/applications 2>/dev/null
```

If the launcher does not immediately appear in GNOME, log out and log back in.

---

# 8. Why This Is Necessary

BURAI is an older JavaFX application originally designed for Java 8/11-era environments.

Fedora 44 uses:

- OpenJDK 25
- Wayland by default
- New Java module system

This causes multiple compatibility problems:

| Problem                  | Solution               |
| ------------------------ | ---------------------- |
| Java 25 incompatibility  | Use Temurin 17         |
| JavaFX version mismatch  | Use JavaFX 17 SDK      |
| Wayland rendering issues | Force X11 backend      |
| Missing desktop launcher | Create `.desktop` file |

The final working stack is:

```text
BURAI 1.3.x
+ Temurin 17
+ JavaFX 17
+ X11 backend
```

This configuration is currently the most stable solution on Fedora 44.


