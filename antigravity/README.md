# Antigravity IDE Setup & Usage Guide (Linux ARM64)

This module installs **Antigravity IDE (v2.6.0 Linux ARM64)** inside your Termux Ubuntu environment and links it to **Termux:X11**, the **`url`** OAuth callback helper, and the **`dev`** web server workflow.

* **Target Architecture:** `linux-arm` (`arm64` / `aarch64`)
* **Prerequisite:** Complete the base setup in the [Main README](../README.md) first and ensure **Termux:X11** is installed on your phone.

---

## 1. One-Command Installation

1. Open **Termux** and enter Ubuntu:
```bash
start os
```
2. Inside **Ubuntu (`root@localhost:~#`)**, paste this single command to install required GTK/X11 libraries, download Antigravity IDE v2.6.0 (`linux-arm`), extract it to `/opt/antigravity`, and link the binary:
```bash
apt update && apt install wget tar libnss3 libatk-bridge2.0-0 libgtk-3-0 libgbm1 libx11-xcb1 xdg-utils -y && (apt install libasound2t64 -y || apt install libasound2 -y) && rm -rf /opt/antigravity && mkdir -p /opt/antigravity && wget -O /tmp/antigravity.tar.gz "https://edgedl.me.gvt1.com/edgedl/release2/j0qc3/antigravity/stable/2.6.0-5526304964083712/linux-arm/Antigravity%20IDE.tar.gz" && tar -xzf /tmp/antigravity.tar.gz -C /opt/antigravity --strip-components=1 && rm -f /tmp/antigravity.tar.gz && chmod +x /opt/antigravity/antigravity 2>/dev/null && ln -sf /opt/antigravity/antigravity /usr/local/bin/antigravity && echo ">> Antigravity IDE installed successfully!"
```

---

## 2. Daily Usage Instructions

### Launch & Stop the IDE (`code` / `stop code`)
* To start Antigravity IDE inside `/sdcard/Documents/Projects` and automatically open **Termux:X11**, run:
```bash
code
```
* To close Antigravity IDE cleanly from the Ubuntu terminal, run:
```bash
stop code
```

### How to Sign In / Authenticate (`url` Shortcut)
When signing into your account in Antigravity IDE redirects your Android browser to a `http://localhost:...` URL that says unreachable:
1. Copy the full `http://localhost:...` URL from your browser address bar.
2. Switch to your Ubuntu terminal and run:
```bash
url
```
3. Paste the localhost URL and press **Enter**. Your IDE is now signed in.

### Running React / Vite / Node Web Projects (`dev` Shortcut)
Place your project files inside `/sdcard/Documents/Projects` and run:
```bash
dev
```
This syncs your source files into native Linux storage (`/root/projects`), installs `node_modules` without `/sdcard` or Node v22 socket freezes, and starts the live server on `http://localhost:5173` or `http://localhost:3000`.
