# Uninstall & Factory Reset Guide

Use these commands to uninstall individual applications to free up space or completely reset the Ubuntu environment.

---

## Part 1: Uninstall Specific Applications Only

Enter Ubuntu first (`start os`), then run the block for the app you wish to remove inside **`root@localhost:~#`**:

### 1. Remove Antigravity IDE Only
```bash
stop code 2>/dev/null
rm -rf /opt/antigravity /usr/local/bin/antigravity ~/.config/Antigravity ~/.antigravity
echo ">> Antigravity IDE has been completely removed."
```

### 2. Remove MATLAB (GNU Octave) Only
```bash
stop matlab 2>/dev/null
apt remove --purge octave* -y && apt autoremove -y && apt clean
rm -f /root/amat2120.m /root/.octaverc
echo ">> GNU Octave (MATLAB) has been completely removed."
```

### 3. Remove Brave Browser Only
```bash
apt remove --purge brave-browser -y && apt autoremove -y && apt clean
rm -f /usr/bin/brave /usr/bin/brave-browser /etc/apt/sources.list.d/brave-browser-release.list
echo ">> Brave Browser has been completely removed."
```

### 4. Clear Native Web Build Cache (`/root/projects`)
```bash
rm -rf /root/projects
echo ">> Native /root/projects build cache deleted (source files in /sdcard/Documents/Projects are safe)."
```

---

## Part 2: Complete Factory Reset (Delete Everything)

To wipe Ubuntu, all Linux apps, VNC settings, and Termux shortcuts, exit Ubuntu (`exit`) back to **Termux (`~ $`)** and run:
```bash
pkill -f termux.x11 2>/dev/null
proot-distro remove ubuntu
sed -i '/# === TERMUX UBUNTU LAUNCHER ===/,/# === END LAUNCHER ===/d' ~/.bashrc
rm -f "$PREFIX/tmp/launch_x11"
source ~/.bashrc
echo ">> Complete reset finished: Ubuntu and all Termux shortcuts removed."
```

*(Note: Your project files inside `/sdcard/Documents/Projects` are kept safe. Run `rm -rf ~/storage/shared/Documents/Projects` only if you want to delete your projects too.)*
