# MATLAB (GNU Octave) Setup & Usage Guide

This module installs **GNU Octave** with full graphical user interface (GUI) support inside Ubuntu so you can write, plot, and execute MATLAB `.m` scripts on your Android phone via **Termux:X11**.

* **Prerequisite:** Complete the base setup in the [Main README](../README.md) first and ensure **Termux:X11** is installed.

---

## 1. One-Command Installation

Log into Ubuntu (`start os`) and paste this command inside **`root@localhost:~#`**:
```bash
apt update && apt install octave dbus-x11 -y && mkdir -p /sdcard/Documents/Projects/MATLAB && cat << 'EOF' > /root/.octaverc
addpath('/root');
EOF
cat << 'EOF' > /root/amat2120.m
cd('/sdcard/Documents/Projects/MATLAB');
disp('==================================================');
disp('>> Switched to MATLAB Workspace: /sdcard/Documents/Projects/MATLAB');
disp('==================================================');
ls
EOF
echo ">> GNU Octave (MATLAB) installed and configured!"
```

---

## 2. Daily Usage Instructions

* **Launch GUI (`matlab`):** Run `matlab` in Ubuntu to automatically open the Octave GUI inside **Termux:X11**.
* **Switch to Coursework Folder (`amat2120`):** Inside the Octave Command Window, type `amat2120` to jump directly to `/sdcard/Documents/Projects/MATLAB` and list your `.m` files.
* **Stop Octave (`stop matlab`):** Run `stop matlab` in the Ubuntu terminal to close Octave.
