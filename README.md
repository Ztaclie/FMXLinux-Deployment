# 📦 Packaging a Delphi FMX Application for Linux (.deb)

This guide provides a complete step-by-step walkthrough for packaging a **Delphi FMX application (built with FMXLinux)** into a `.deb` installer that works on Debian-based Linux distributions such as Ubuntu, Linux Mint, Pop!_OS, etc.

The resulting `.deb` file:
- Installs your app to system folders
- Adds a launcher icon to the app menu
- Includes custom libraries like `libsk4d.so` (if needed)
- Can be double-clicked or installed via terminal

---

## 🧱 Folder Structure for `.deb` Packaging

Your packaging workspace should follow this structure:

```
<app-name>-1.0/
├── DEBIAN/
│   └── control
└── usr/
    ├── bin/
    │   └── <app-name>            # Launcher script
    ├── lib/
    │   └── <app-name>/
    │       ├── <AppBinary>       # Your compiled FMX binary
    │       └── <Library>.so      # Any required custom libraries
    └── share/
        ├── applications/
        │   └── <app-name>.desktop
        └── icons/
            └── hicolor/
                └── 512x512/
                    └── apps/
                        └── <app-name>.png
```

---

## ⚙️ Step-by-Step Instructions

### 1. Create the Build Directory

```bash
mkdir -p ~/deb-build/<app-name>-1.0
cd ~/deb-build/<app-name>-1.0
```

---

### 2. Create the Control File

Create the `DEBIAN/control` file that defines metadata for the package.

```bash
mkdir DEBIAN
nano DEBIAN/control
```

Paste this template:

```plaintext
Package: <app-name>
Version: 1.0
Section: utils
Priority: optional
Architecture: amd64
Depends: libc6 (>= 2.29)
Maintainer: Your Name <you@example.com>
Description: <App_Name> - A cross-platform application built with Delphi FMX.
 A native Linux desktop app using FMXLinux.
```

---

### 3. Copy App Binary and Libraries

Create the library folder:

```bash
mkdir -p usr/lib/<app-name>
```

Copy your app binary and any required libraries:

```bash
cp /path/to/<Your_Binary> usr/lib/<app-name>/
cp /path/to/<Any_Required_Library>.so usr/lib/<app-name>/
```

---

### 4. Create the Launcher Script

Instead of launching the binary directly, create a shell script to wrap it with the correct environment.

```bash
mkdir -p usr/bin
nano usr/bin/<app-name>
```

Paste this script:

```bash
#!/bin/bash
export LD_LIBRARY_PATH=/usr/lib/<app-name>:$LD_LIBRARY_PATH
exec /usr/lib/<app-name>/<Your_Binary> "$@"
```

Then make it executable:

```bash
chmod +x usr/bin/<app-name>
```

---

### 5. Create Desktop Entry

Create a desktop entry to integrate your app into the Linux UI (menu, search, taskbar):

```bash
mkdir -p usr/share/applications
nano usr/share/applications/<app-name>.desktop
```

Paste this:

```ini
[Desktop Entry]
Version=1.0
Name=<App_Name>
Comment=A cross-platform application built with Delphi FMX
Exec=<app-name>
Icon=<app-name>
Terminal=false
Type=Application
Categories=Utility;
StartupNotify=true
StartupWMClass=<App_Name>
```

---

### 6. Add Application Icon

Create the icon path and copy a 512x512 PNG icon:

```bash
mkdir -p usr/share/icons/hicolor/512x512/apps
cp /path/to/<Your_Icon>.png usr/share/icons/hicolor/512x512/apps/<app-name>.png
```

---

### 7. Build the `.deb` Package

From the parent folder:

```bash
cd ~/deb-build
dpkg-deb --build <app-name>-1.0
```

You will get:

```
<app-name>-1.0.deb
```

---

## ✅ Optional: Using `LD_LIBRARY_PATH` for Custom Libraries

### Why It's Needed

If your FMX application uses shared libraries (like `libsk4d.so`, third-party graphics/audio libs, etc.) and they are not installed system-wide, the app may crash with:

```
error while loading shared libraries: libxyz.so: cannot open shared object file: No such file or directory
```

### The Solution

The launcher script sets a temporary library path using `LD_LIBRARY_PATH`, like this:

```bash
export LD_LIBRARY_PATH=/usr/lib/<app-name>:$LD_LIBRARY_PATH
```

This tells the system to look in your app’s private directory for any required `.so` files — no system-wide installation required.

---

## 🚀 Installing the `.deb` File

### Option A: Terminal

```bash
sudo dpkg -i <app-name>-1.0.deb
sudo apt-get install -f  # Fix any dependency issues if needed
```

### Option B: GUI

- Double-click the `.deb` file
- Install it via Software Center or GDebi

---

## 🔄 Uninstalling the App

```bash
sudo apt remove <app-name>
```

This will clean up the binary, desktop entry, icon, and libraries.

---

## 🧪 Troubleshooting Tips

- **App doesn't launch?** Run it from the terminal to check for errors:
  ```bash
  <app-name>
  ```
- **Library not found?** Ensure it's included in `/usr/lib/<app-name>` and `LD_LIBRARY_PATH` is set in your launcher script.
- **No icon or launcher?** Double-check `.desktop` file paths and that it’s placed in `/usr/share/applications`.

---

## 🙋 Need Help?

If you're using Delphi + FMXLinux and need help automating this process or building AppImages, feel free to reach out or look into:

- [`dpkg-deb`](https://manpages.debian.org/bullseye/dpkg-dev/dpkg-deb.1.en.html)
- [`LD_LIBRARY_PATH` documentation](https://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html)

---

## 🧰 Tools Used

- Delphi + FMXLinux (for app build)
- `dpkg-deb` (to create the `.deb`)
- Bash scripting (launcher)
- `gtk-launch`, `desktop-file-validate` for testing

---
