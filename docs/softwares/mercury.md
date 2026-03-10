---
sidebar_position: 3
---
# Mercury

> For ubuntu 24.04 LTS

A crystal editing software from CCDC

## 1️⃣ Download and install

Go to the offical website: <https://www.ccdc.cam.ac.uk/solutions/software/free-mercury/>

Sign up and download the installer.

```bash
cd ~/Downloads
chmod +x CSDInstallerOnline-2025.3.1-linux
sudo ./CSDInstallerOnline-2025.3.1-linux
```

Then follow the install step, keep the default install directory or choose the directory you want to install.

After installation, you will get some directory like below

```bash
├── CCDC
│  ├── ccdc-software
│  ├── ccdc-utilities
│  ├── installerResources
│  ├── Licenses
│  ├── tmpMaintenanceToolApp
│  ├── ccdc-maintenance-tool
│  ├── ccdc-maintenance-tool.dat
│  ├── ccdc-maintenance-tool.new
│  ├── components.xml
│  ├── InstallationLog.txt
│  ├── installer.dat
│  └── network.xml
```

Now Mercury has been installed in `ccdc-software/mercury`

## 2️⃣ Wayland Desktop

If your desktop eviroment is wayland, you will also need

```bash
export QT_QPA_PLATFORM=xcb ./path/to/mercury
```

## 3️⃣ Create a Desktop Launcher (Optional)

Create a new file in `/home/USERNAME/.local/share/applications`

```bash
vi $HOME/.local/share/applications/mercury.desktop
```

```bash
[Desktop Entry]
Name=Mercury
Comment=CCDC Mercury Crystal Structure Visualizer
Exec=env CSDHOME=$HOME/CCDC QT_QPA_PLATFORM=xcb $HOME/CCDC/ccdc-software/mercury/bin/mercury
Icon=$HOME/CCDC/ccdc-software/mercury/icons/mercury_48x48.png
Terminal=false
Type=Application
Categories=Science;Chemistry;
StartupNotify=true
```

## 4️⃣ Create a CLI Launcher

Check if there is `export PATH="$HOME/bin:$PATH"` in `~/.zshrc` or `~/.bashrc` (depend on the shell you are using).

Then create a script in `~/bin`

```bash
vi ~/bin/mercury
```

```bash
#!/usr/bin/env bash
export CSDHOME="$HOME/CCDC"
export QT_QPA_PLATFORM=xcb
exec "$HOME/CCDC/ccdc-software/mercury/bin/mercury" "$@"
```

Then

```bash
mercury
```

Enjoy the software.
