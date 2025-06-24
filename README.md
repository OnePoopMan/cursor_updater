# A Guide to Automating Cursor IDE Updates on Linux

This guide provides a simple way to automate the setup and updating of the Cursor IDE AppImage on Linux.

## Quick Setup

You can automate the entire setup process by running the following script.

### 1. Create the Setup Script

Copy and paste the entire contents of the code block below into a new file named `setup.sh`.

```bash
#!/bin/bash

# This script automates the setup of Cursor IDE for auto-updating and desktop integration.

echo "Starting Cursor IDE setup..."

# Step 1: Create necessary directories
echo "Creating directories..."
mkdir -p ~/Applications/cursor
mkdir -p ~/.config/systemd/user
mkdir -p ~/.local/share/applications

# Step 2: Create the update script
echo "Creating update script..."
cat << 'EOF' > ~/Applications/cursor/update-cursor.sh
#!/bin/bash

APPDIR=~/Applications/cursor
APPIMAGE_URL="https://www.cursor.com/download/stable/linux-x64"

wget -O $APPDIR/cursor.AppImage $APPIMAGE_URL
chmod +x $APPDIR/cursor.AppImage
EOF

# Step 3: Make the update script executable
echo "Making update script executable..."
chmod +x ~/Applications/cursor/update-cursor.sh

# Step 4: Create the systemd service file
echo "Creating systemd service..."
cat << EOF > ~/.config/systemd/user/update-cursor.service
[Unit]
Description=Update Cursor IDE

[Service]
ExecStart=/home/$USER/Applications/cursor/update-cursor.sh
Type=oneshot

[Install]
WantedBy=default.target
EOF

# Step 5: Enable and start the systemd service
echo "Enabling and starting systemd service..."
systemctl --user enable update-cursor.service
systemctl --user start update-cursor.service

# Step 6: Create the desktop entry
echo "Creating desktop entry..."
cat << EOF > ~/.local/share/applications/cursor.desktop
[Desktop Entry]
Name=Cursor
Exec=/home/$USER/Applications/cursor/cursor.AppImage
Icon=/home/$USER/Applications/cursor/cursor-icon.png
Type=Application
Categories=Utility;Development;
StartupWMClass=Cursor
EOF

# Step 7: Download the icon
echo "Downloading icon..."
wget -O ~/Applications/cursor/cursor-icon.png https://avatars.githubusercontent.com/u/126759922

echo ""
echo "Setup complete!"
echo "The script has run the first update for you and downloaded the icon."
echo "You should now find Cursor in your application menu."
```

### 2. Make the Script Executable

Open a terminal and run the following command:

```bash
chmod +x setup.sh
```

### 3. Run the Script

Execute the script with:

```bash
./setup.sh
```

The script will handle everything: creating directories, setting up the update script, creating the systemd service for auto-updates, and adding the desktop entry for application menu integration.

### 4. Add the Icon

The only manual step is to download an icon for Cursor.
- Download your preferred icon. A good one can be found at [Cursor's Github Avatar](https://avatars.githubusercontent.com/u/126759922).
- - Save it as `cursor-icon.png` in the `~/Applications/cursor/` folder.

## Troubleshooting

### `systemctl --user` Errors

If you encounter errors with `systemctl --user`, here are some common fixes.

First, check the status of systemd user services:

```bash
systemctl --user status
```

It should show `State: running`.

If you see an error like `Failed to connect to bus: Connection refused`, try starting the dbus service manually:

```bash
eval $(dbus-launch --sh-syntax)
```

Then try your `systemctl --user` command again.

If your shell environment is not correctly configured, you might need to manually set up your environment variables. Add the following lines to your `.zshrc` or `.bashrc` file:

```bash
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
```

After adding these lines, reload your shell configuration:

```bash
source ~/.zshrc # or source ~/.bashrc
```

If `dbus-launch` is not installed, install it with:

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt install dbus-x11
```

Then restart your system.

### Toolbar Icon Not Showing

If the Cursor icon appears in your application menu but not in the toolbar when the application is running, it's likely an issue with the `StartupWMClass` entry in the `.desktop` file.

The setup script already includes `StartupWMClass=Cursor`, which works for most systems. If you are still having issues, you can find the correct window class for your system:

1.  **Find the Window Class:**
    Make sure Cursor is running. Then, open a terminal and run:
    ```bash
    xprop WM_CLASS
    ```
    Your cursor will turn into a crosshair. Click on the main Cursor application window (not the toolbar icon). The output will be something like `WM_CLASS(STRING) = "cursor", "Cursor"`. The second string, "Cursor", is the one we need.

2.  **Update the `.desktop` file:**
    Open your `cursor.desktop` file:
    ```bash
    vim ~/.local/share/applications/cursor.desktop
    ```
    And replace `Cursor` in `StartupWMClass=Cursor` with the value you found.

## Conclusion

You have successfully set up an automated update process for Cursor IDE on your Linux machine. Enjoy the latest features of AI-assisted coding without the hassle of manual updates!
