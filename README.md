# Termux Ubuntu Dev Environment (Antigravity IDE + Web Dev + MATLAB/Octave)

A complete, native-speed **Ubuntu Linux (ARM64)** development environment for Android powered by **Termux**, **Termux:X11**, and **AVNC**.

---

## Key Features & Fixes

1. **Android `/sdcard` FUSE Bypass (`dev` command):** Keeps your editable code in `/sdcard/Documents/Projects` while automatically syncing and running `node_modules` inside native Linux storage (`/root/projects`) so `npm install` and Vite (`esbuild`) never fail.
2. **Node.js v22 PRoot Network Freeze Fix:** Disables broken IPv6 family auto-selection and caps concurrent sockets so `npm install` never hangs.
3. **Automatic GUI App Switching:** Typing `code` or `matlab` automatically opens **Termux:X11** and displays the GUI window.
4. **Pre-Patched Brave Browser Sandbox:** Configures Brave Browser (`arm64`) with `--no-sandbox --disable-gpu --disable-dev-shm-usage`.
5. **One-Word OAuth Callback Helper (`url`):** Completes localhost OAuth logins directly from the terminal.

---

## 1. Required Android Apps (Install First)

| App Name | Download Source | Purpose |
| :--- | :--- | :--- |
| **1. Termux** | [F-Droid](https://f-droid.org/en/packages/com.termux/) or [GitHub Releases](https://github.com/termux/termux-app/releases) | Core Linux terminal. **Do not** use the Play Store version. |
| **2. Termux:X11** | [GitHub Releases (APK)](https://github.com/termux/termux-x11/releases) | Renders hardware-accelerated Linux GUI windows (`arm64-v8a` APK). |
| **3. AVNC** | [F-Droid](https://f-droid.org/en/packages/com.gaurav.avnc/) or Play Store | VNC client to view the **XFCE4 Desktop** and **Brave Browser** on port `5901`. |

---

## 2. Base Termux & Ubuntu Setup

Open **Termux** (`~ $`) on your phone and run the steps below in order.

### Step 1: Grant Storage Access & Install Core Packages
Tap **Allow** when Android asks for storage permission:
```bash
termux-setup-storage
pkg update && pkg upgrade -y
pkg install x11-repo -y
pkg install termux-x11-nightly pulseaudio proot-distro wget curl git -y
proot-distro install ubuntu
```

### Step 2: Configure Termux Shortcuts (`start os` & `dev`)
Paste this block into **Termux (`~ $`)** to set up `start os` (with the Termux:X11 auto-launcher) and `dev`:
```bash
sed -i '/# === TERMUX UBUNTU LAUNCHER ===/,/# === END LAUNCHER ===/d' ~/.bashrc
cat << 'EOF' >> ~/.bashrc
# === TERMUX UBUNTU LAUNCHER ===
start() {
    if [ "$1" = "os" ]; then
        mkdir -p ~/storage/shared/Documents/Projects
        termux-x11 :0 -ac > /dev/null 2>&1 &
        ( while true; do
            if [ -f "$PREFIX/tmp/launch_x11" ]; then
                rm -f "$PREFIX/tmp/launch_x11"
                am start -n com.termux.x11/com.termux.x11.MainActivity > /dev/null 2>&1
            fi
            sleep 0.5
        done ) &
        local watcher_pid=$!
        proot-distro login ubuntu --shared-tmp
        kill $watcher_pid 2>/dev/null
    else
        echo ">> Usage: start os"
    fi
}

dev() {
    proot-distro login ubuntu --shared-tmp -- bash -ic "dev $1"
}
# === END LAUNCHER ===
EOF
source ~/.bashrc
```

---

## 3. Inside Ubuntu: Desktop, Brave Browser & Control Center

Enter Ubuntu from Termux:
```bash
start os
```

### Step 1: Install XFCE4 Desktop, VNC, Node.js, GitHub CLI & Brave Browser
Paste this inside **Ubuntu (`root@localhost:~#`)**:
```bash
apt update && apt upgrade -y
apt install sudo nano wget curl git xfce4 xfce4-goodies tigervnc-standalone-server dbus-x11 x11-xserver-utils libnss3 libatk-bridge2.0-0 libgtk-3-0 libgbm1 libasound2t64 libx11-xcb1 xdg-utils nodejs npm gh -y || apt install libasound2 -y

curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg arch=arm64] https://brave-browser-apt-release.s3.brave.com/ stable main" > /etc/apt/sources.list.d/brave-browser-release.list
apt update && apt install brave-browser -y
printf "#!/bin/bash\nexec /opt/brave.com/brave/brave --no-sandbox --disable-gpu --disable-dev-shm-usage \"\$@\"\n" > /usr/bin/brave
chmod +x /usr/bin/brave
ln -sf /usr/bin/brave /usr/bin/brave-browser
```

### Step 2: Install the Ubuntu Linux Control Center
Paste this inside **Ubuntu (`root@localhost:~#`)** to configure all shortcuts (`code`, `matlab`, `start server`, `url`, `dev`, `glogin`, `gpull`, `gpush`):
```bash
sed -i '/# === UBUNTU CONTROL CENTER ===/,/# === END CONTROL CENTER ===/d' /root/.bashrc
cat << 'EOF' >> /root/.bashrc
# === UBUNTU CONTROL CENTER ===
export DISPLAY=:0
export PULSE_SERVER=127.0.0.1

code() {
    echo ">> Launching Antigravity IDE in: /sdcard/Documents/Projects"
    echo ">> To stop it, run: stop code"
    mkdir -p /sdcard/Documents/Projects
    touch /tmp/launch_x11
    DISPLAY=:0 /opt/antigravity/antigravity --no-sandbox /sdcard/Documents/Projects > /dev/null 2>&1 &
}

matlab() {
    echo ">> Launching MATLAB (Octave)..."
    echo ">> To stop it, run: stop matlab"
    touch /tmp/launch_x11
    DISPLAY=:0 dbus-launch octave --gui > /dev/null 2>&1 &
}

start() {
    if [ "$1" = "server" ]; then
        mkdir -p ~/.vnc
        printf "#!/bin/bash\nexec startxfce4\n" > ~/.vnc/xstartup
        chmod +x ~/.vnc/xstartup
        vncserver -kill :1 > /dev/null 2>&1
        vncserver :1 -geometry 1280x720 -depth 24 -localhost no -SecurityTypes None --I-KNOW-THIS-IS-INSECURE
        echo ">> VNC Server started on :1 (Port 5901). Open AVNC and connect to localhost:5901"
    else
        echo ">> Usage: start server"
    fi
}

stop() {
    if [ "$1" = "code" ]; then
        pkill -f antigravity && echo ">> Antigravity IDE stopped."
    elif [ "$1" = "matlab" ]; then
        pkill -f octave && echo ">> MATLAB (Octave) stopped."
    elif [ "$1" = "server" ]; then
        vncserver -kill :1 && echo ">> VNC Server stopped."
    else
        echo ">> Usage: stop code | stop matlab | stop server"
    fi
}

url() {
    read -p ">> Paste the localhost callback URL here: " auth_url
    if [ -n "$auth_url" ]; then
        curl -L "$auth_url" > /dev/null 2>&1 && echo ">> Authentication callback sent successfully!"
    else
        echo ">> Error: No URL provided."
    fi
}

dev() {
    mkdir -p /root/projects
    tar -cf - --exclude="node_modules" --exclude=".git" -C /sdcard/Documents/Projects . | tar -xf - -C /root/projects
    cd /root/projects
    if [ ! -d "node_modules" ]; then
        echo ">> Installing dependencies in native Linux storage..."
        NODE_OPTIONS="--no-network-family-autoselection --dns-result-order=ipv4first" npm install --no-audit --no-fund --maxsockets=3
    fi
    echo ">> Starting dev server on http://localhost:5173 or http://localhost:3000 (Ctrl+C to stop)"
    npm run dev -- --host
}

glogin() {
    gh auth login
}

gpull() {
    cd /sdcard/Documents/Projects && git pull && tar -cf - --exclude="node_modules" --exclude=".git" -C /sdcard/Documents/Projects . | tar -xf - -C /root/projects
    echo ">> Pulled latest changes from GitHub and synced!"
}

gpush() {
    if [ -z "$1" ]; then echo ">> Usage: gpush \"commit message\""; return 1; fi
    cd /sdcard/Documents/Projects && git add . && git commit -m "$1" && git push
}

echo "=================================================="
echo "           UBUNTU LINUX CONTROL CENTER"
echo "=================================================="
echo "  1. Antigravity IDE:   >> code | stop code"
echo "  2. MATLAB (Octave):   >> matlab | stop matlab"
echo "     (Inside Octave, type: amat2120)"
echo "  3. VNC Server:        >> start server | stop server"
echo "  4. Auth Helper:       >> url"
echo "  5. Web Dev Server:    >> dev"
echo "  6. GitHub Sync:       >> glogin | gpull | gpush \"msg\""
echo "=================================================="
# === END CONTROL CENTER ===
EOF
source /root/.bashrc
```

---

## 4. Shortcut Reference & Application Modules

| Command | Where to Run | Action |
| :--- | :--- | :--- |
| `start os` | Termux (`~ $`) | Starts Termux:X11 display server and logs into Ubuntu. |
| `code` / `stop code` | Ubuntu | Launches or stops **Antigravity IDE** in `/sdcard/Documents/Projects` via Termux:X11. |
| `matlab` / `stop matlab` | Ubuntu | Launches or stops **GNU Octave GUI** via Termux:X11. |
| `start server` / `stop server` | Ubuntu | Starts or stops XFCE4 VNC desktop on `localhost:5901` for **AVNC**. |
| `url` | Ubuntu | Completes `localhost` OAuth redirect sign-in for the IDE. |
| `dev` | Termux or Ubuntu | Syncs `/sdcard/Documents/Projects` to `/root/projects`, installs npm packages, and starts Vite/Node. |
| `glogin` / `gpull` / `gpush "msg"` | Ubuntu | Logs into GitHub CLI, pulls repo updates, or commits and pushes changes. |

### Application & Maintenance Guides
* [**Antigravity IDE Setup (`./antigravity/README.md`)**](./antigravity/README.md)
* [**MATLAB / GNU Octave Setup (`./octave/README.md`)**](./octave/README.md)
* [**Uninstall & Full Reset Guide (`./uninstall/README.md`)**](./uninstall/README.md)

---
*Created & Maintained by Hasan Shahriar Adib*
