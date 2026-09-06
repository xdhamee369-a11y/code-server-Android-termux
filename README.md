# Code Server on Android Termux

Run an open-source VS Code-like development environment directly on an Android phone using **Termux + Ubuntu (PRoot) + code-server**.

This setup is suitable for Android devices such as the OnePlus 10R and other reasonably powerful phones.

<p align="center">
  <img
    src="https://raw.githubusercontent.com/github/explore/main/topics/android/android.png"
    width="100"
    alt="Android"
  >
</p>

<p align="center"> <img src="https://img.shields.io/badge/Android-Termux-3DDC84?style=flat-square&logo=android&logoColor=white" alt="Android"> <img src="https://img.shields.io/badge/Code--Server-Open%20Source-18181B?style=flat-square" alt="Code Server"> <img src="https://img.shields.io/badge/Ubuntu-PRoot-E95420?style=flat-square&logo=ubuntu&logoColor=white" alt="Ubuntu"> <img src="https://img.shields.io/badge/License-MIT-18181B?style=flat-square" alt="MIT License"> </p>

<p align="center"> <b>Turn your Android phone into a portable development machine.</b> </p>

Preview

<p align="center"> <img src="https://raw.githubusercontent.com/coder/code-server/main/docs/images/code-server.png" alt="code-server" width="850"> </p>

## What you get

* Termux-based Linux development environment
* Ubuntu userspace through `proot-distro`
* `code-server` in a browser
* Git
* Node.js
* Python
* Build tools
* Persistent project files
* Simple start/stop/status scripts

## Architecture

```text
Android
   │
   └── Termux
        │
        └── proot-distro
             │
             └── Ubuntu
                  │
                  ├── code-server
                  │
                  ├── Git
                  ├── Node.js
                  └── Python
                       │
                       ▼
                Browser :8080
```

## Requirements

* Android phone
* Termux
* Internet connection
* Several GB of free storage recommended
* A modern browser

> Install Termux from a trusted source such as F-Droid or the official Termux project rather than an outdated Play Store build.

---

# 1. Install Termux

Open Termux and update the package repository:

```bash
pkg update
pkg upgrade -y
```

Give Termux access to Android shared storage:

```bash
termux-setup-storage
```

Grant the permission when Android asks.

You should then have access to:

```text
~/storage/shared
```

---

# 2. Install required Termux packages

```bash
pkg install -y \
  git \
  curl \
  wget \
  nano \
  proot \
  proot-distro \
  tar \
  unzip
```

Check that `proot-distro` works:

```bash
proot-distro --version
```

---

# 3. Install Ubuntu

```bash
proot-distro install ubuntu
```

Enter Ubuntu:

```bash
proot-distro login ubuntu
```

You are now inside the Ubuntu userspace.

Check it:

```bash
cat /etc/os-release
```

---

# 4. Update Ubuntu

Inside Ubuntu:

```bash
apt update
apt upgrade -y
```

Install common development tools:

```bash
apt install -y \
  curl \
  wget \
  git \
  nano \
  build-essential \
  python3 \
  python3-pip \
  ca-certificates
```

---

# 5. Install code-server

Inside Ubuntu:

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

Verify the installation:

```bash
code-server --version
```

If the command is available, installation succeeded.

---

# 6. Start code-server

For a local-only server:

```bash
code-server \
  --bind-addr 127.0.0.1:8080 \
  --auth none
```

Open your Android browser:

```text
http://127.0.0.1:8080
```

You should see the code-server interface.

---

# 7. Recommended authentication setup

Do **not** use `--auth none` if you intend to expose code-server beyond your phone.

Create the configuration directory:

```bash
mkdir -p ~/.config/code-server
```

Create the configuration file:

```bash
nano ~/.config/code-server/config.yaml
```

Example:

```yaml
bind-addr: 127.0.0.1:8080
auth: password
password: CHANGE_THIS_PASSWORD
cert: false
```

Then start:

```bash
code-server
```

Access:

```text
http://127.0.0.1:8080
```

---

# 8. Create a workspace

Create a development directory:

```bash
mkdir -p ~/projects
```

Enter it:

```bash
cd ~/projects
```

Start code-server with that directory:

```bash
code-server ~/projects
```

Now your Android phone becomes a portable development environment.

---

# 9. Access Android shared storage

Termux exposes Android shared storage through:

```text
~/storage/shared
```

Depending on how the Ubuntu PRoot environment is configured, Android storage may not appear automatically inside Ubuntu.

A simple approach is to create a workspace in Termux-accessible storage and bind/mount it into the Ubuntu environment as appropriate for your setup.

For example, from Termux:

```bash
mkdir -p ~/storage/shared/code
```

Then use that directory for projects when accessible from your Ubuntu environment.

---

# 10. Start Ubuntu from Termux

Exit Ubuntu:

```bash
exit
```

From Termux:

```bash
proot-distro login ubuntu
```

Then:

```bash
code-server --bind-addr 127.0.0.1:8080
```

Open:

```text
http://127.0.0.1:8080
```

---

# 11. One-command launcher

Create:

```bash
nano ~/start-code-server.sh
```

Add:

```bash
#!/data/data/com.termux/files/usr/bin/bash

set -e

echo "Starting Ubuntu..."
echo "Starting code-server on http://127.0.0.1:8080"

proot-distro login ubuntu -- bash -lc '
    code-server \
        --bind-addr 127.0.0.1:8080
'
```

Make it executable:

```bash
chmod +x ~/start-code-server.sh
```

Start the server:

```bash
~/start-code-server.sh
```

---

# 12. Background mode

If you want the server to continue while you work in Termux:

```bash
nohup proot-distro login ubuntu -- \
  bash -lc 'code-server --bind-addr 127.0.0.1:8080' \
  > ~/code-server.log 2>&1 &
```

Check the log:

```bash
tail -f ~/code-server.log
```

---

# 13. Stop code-server

Find the process:

```bash
pgrep -af code-server
```

Stop it:

```bash
pkill -f code-server
```

If you are running it interactively, `Ctrl+C` is usually enough.

---

# 14. Check server status

```bash
curl -I http://127.0.0.1:8080
```

A running server should return an HTTP response.

You can also check:

```bash
pgrep -af code-server
```

---

# 15. Performance tips for Android

Android aggressively manages background applications.

For long-running development sessions:

1. Open Android Settings.
2. Find the battery settings for Termux.
3. Disable battery optimization for Termux where your Android version provides that option.
4. Keep Termux running when using code-server.
5. Use a Termux wakelock when you specifically need the device to remain awake.

Inside Termux, if the Termux API/wakelock functionality is available:

```bash
termux-wake-lock
```

Release it when finished:

```bash
termux-wake-unlock
```

Do not keep a wakelock active unnecessarily because it can increase battery consumption.

---

# 16. Useful commands

### Enter Ubuntu

```bash
proot-distro login ubuntu
```

### Exit Ubuntu

```bash
exit
```

### Start code-server

```bash
code-server --bind-addr 127.0.0.1:8080
```

### Start without authentication

```bash
code-server --bind-addr 127.0.0.1:8080 --auth none
```

Only use this for a deliberately local-only setup.

### Check code-server

```bash
code-server --version
```

### Check Ubuntu

```bash
cat /etc/os-release
```

### Check processes

```bash
ps aux | grep code-server
```

---

# 17. Troubleshooting

## `code-server: command not found`

Check whether it was installed:

```bash
which code-server
```

Try:

```bash
command -v code-server
```

If installation failed, rerun:

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

---

## Port 8080 is already in use

Check:

```bash
ss -ltnp | grep 8080
```

Use another port:

```bash
code-server --bind-addr 127.0.0.1:8081
```

Then open:

```text
http://127.0.0.1:8081
```

---

## Android kills Termux

Check Android's battery/background restrictions for Termux.

Also consider using:

```bash
termux-wake-lock
```

while actively running a long development session.

---

## code-server is slow

PRoot introduces overhead compared with a normal Linux installation.

Reduce the workload:

* Close unused browser tabs.
* Avoid running unnecessary development servers.
* Avoid huge repositories.
* Avoid indexing massive directories.
* Keep projects on reasonably fast storage.
* Use a local-only bind address.
* Close code-server when you aren't using it.

---

# 18. Security

The safest default is:

```bash
code-server --bind-addr 127.0.0.1:8080
```

This keeps the server accessible from the phone itself.

Avoid exposing:

```text
0.0.0.0:8080
```

unless you understand the networking and authentication implications.

Never assume that `--auth none` is safe simply because you're running it on Android.

---

# 19. Uninstall

Remove Ubuntu:

```bash
proot-distro remove ubuntu
```

Remove the Termux packages:

```bash
pkg uninstall proot-distro
```

Remove your launcher:

```bash
rm -f ~/start-code-server.sh
```

---

# Quick Start

After everything is installed:

```bash
proot-distro login ubuntu
```

Then:

```bash
code-server --bind-addr 127.0.0.1:8080
```

Open:

```text
http://127.0.0.1:8080
```

That's it.

## Recommended setup

For reliability, use:

```text
Termux
  ↓
PRoot-Distro
  ↓
Ubuntu
  ↓
code-server
  ↓
Chrome/Firefox
```

rather than depending on a native npm installation of code-server inside Termux.
