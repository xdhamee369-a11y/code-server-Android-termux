# Code Server on Android Termux

> Run an open-source VS Code-like development environment directly on Android using **Termux**.
> 

1. **Method 1 — Native Termux**: Termux → Node.js → code-server
2. **Method 2 — Ubuntu/PRoot**: Termux → Ubuntu → code-server


<p align="center">
  <img
    src="https://img.shields.io/badge/Android-Termux-3DDC84?style=flat-square&logo=android&logoColor=white"
    alt="Android"
  />
  <img
    src="https://img.shields.io/badge/Code--Server-Open%20Source-18181B?style=flat-square"
    alt="Code Server"
  />
  <img
    src="https://img.shields.io/badge/Ubuntu-PRoot-E95420?style=flat-square&logo=ubuntu&logoColor=white"
    alt="Ubuntu"
  />
  <img
    src="https://img.shields.io/badge/License-MIT-18181B?style=flat-square"
    alt="MIT License"
  />
</p>

<p align="center">
  <b>Turn your Android phone into a portable development machine.</b>
</p>

---

## Preview

<p align="center">
  <img
    src="https://raw.githubusercontent.com/coder/code-server/main/docs/images/code-server.png"
    alt="code-server"
    width="850"
  />
</p>

---

## Choose Your Setup

There are **two ways** to run code-server on Android.

| Method | Setup | Difficulty | Compatibility |
|---|---|---:|---:|
| **Method 1** | Termux + Node.js + code-server | Easy | Good |
| **Method 2** | Termux + Ubuntu + code-server | Medium | Better |

### Method 1 — Native Termux

```text
Android
   │
   ▼
Termux
   │
   ├── Node.js
   │
   └── code-server
            │
            ▼
      Browser :8080
````

Best if you want the **simplest and fastest setup**.

### Method 2 — Ubuntu via PRoot

```text
Android
   │
   ▼
Termux
   │
   ▼
PRoot-Distro
   │
   ▼
Ubuntu
   │
   └── code-server
            │
            ▼
      Browser :8080
```

Recommended if you want a more traditional Linux environment and better compatibility with Linux development tools.

---

# Requirements

* Android phone
* Termux
* Internet connection
* Modern web browser
* Several GB of free storage recommended
* 4 GB+ RAM recommended for a comfortable experience

> Termux should be installed from a current, trusted source. Avoid outdated Termux builds.

---

# Method 1 — Native Termux

This method installs **Node.js and code-server directly inside Termux**.

## 1. Update Termux

```bash
pkg update && pkg upgrade -y
```

Grant storage access:

```bash
termux-setup-storage
```

Allow the Android storage permission when prompted.

---

## 2. Install Node.js and dependencies

```bash
pkg install -y \
  nodejs-lts \
  python \
  make \
  clang \
  git
```

Check Node.js:

```bash
node --version
```

Check npm:

```bash
npm --version
```

---

## 3. Install code-server

```bash
npm install -g code-server
```

Verify:

```bash
code-server --version
```

---

## 4. Start code-server

For local-only access:

```bash
code-server \
  --bind-addr 127.0.0.1:8080
```

Open your Android browser:

```text
http://127.0.0.1:8080
```

You should now see the code-server interface.

---

## 5. Start without authentication

For a local-only development session, you can use:

```bash
code-server \
  --bind-addr 127.0.0.1:8080 \
  --auth none
```

Then open:

```text
http://127.0.0.1:8080
```

> `--auth none` should only be used when the server is intentionally restricted to localhost.

---

## 6. Create a project directory

```bash
mkdir -p ~/projects
cd ~/projects
```

Start code-server in the project directory:

```bash
code-server ~/projects
```

---

# Method 2 — Ubuntu via PRoot

This method runs code-server inside an Ubuntu userspace.

It is generally the better choice when you need a more complete Linux development environment.

## 1. Install PRoot-Distro

From Termux:

```bash
pkg update && pkg upgrade -y
```

Install the required packages:

```bash
pkg install -y \
  proot \
  proot-distro \
  curl \
  git \
  wget \
  nano
```

---

## 2. Install Ubuntu

```bash
proot-distro install ubuntu
```

Enter Ubuntu:

```bash
proot-distro login ubuntu
```

Verify the environment:

```bash
cat /etc/os-release
```

---

## 3. Update Ubuntu

Inside Ubuntu:

```bash
apt update && apt upgrade -y
```

Install development tools:

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

## 4. Install code-server

Inside Ubuntu:

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

Verify:

```bash
code-server --version
```

---

## 5. Start code-server

```bash
code-server \
  --bind-addr 127.0.0.1:8080
```

Open your Android browser:

```text
http://127.0.0.1:8080
```

---

## 6. Create a workspace

Inside Ubuntu:

```bash
mkdir -p ~/projects
cd ~/projects
```

Start code-server:

```bash
code-server ~/projects
```

---

# Authentication

For local development, the default configuration is usually enough.

To configure a password manually:

```bash
mkdir -p ~/.config/code-server
```

Create:

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

Start:

```bash
code-server
```

Then open:

```text
http://127.0.0.1:8080
```

---

# Background Mode

To run code-server in the background from Termux:

```bash
nohup code-server \
  --bind-addr 127.0.0.1:8080 \
  > ~/code-server.log 2>&1 &
```

View logs:

```bash
tail -f ~/code-server.log
```

---

# Stop code-server

Check the process:

```bash
pgrep -af code-server
```

Stop it:

```bash
pkill -f code-server
```

If running interactively, press:

```text
Ctrl + C
```

---

# Android Performance

Android may terminate background processes to save battery.

For longer development sessions:

1. Disable battery optimization for Termux.
2. Allow Termux to run in the background.
3. Keep unnecessary browser tabs closed.
4. Avoid indexing extremely large repositories.
5. Use a wakelock when you need the device to stay awake.

Acquire a wakelock:

```bash
termux-wake-lock
```

Release it when finished:

```bash
termux-wake-unlock
```

> Wakelocks increase battery consumption. Use them only when necessary.

---

# Storage

Termux shared storage is available at:

```text
~/storage/shared
```

After running:

```bash
termux-setup-storage
```

You can create a project directory:

```bash
mkdir -p ~/storage/shared/projects
```

Your Android shared storage can then be used for files that need to be accessible from Android applications.

---

# Troubleshooting

## code-server command not found

Check:

```bash
which code-server
```

Or:

```bash
command -v code-server
```

Check the installation:

```bash
code-server --version
```

---

## Port 8080 is already in use

Check:

```bash
ss -ltnp | grep 8080
```

Use another port:

```bash
code-server \
  --bind-addr 127.0.0.1:8081
```

Open:

```text
http://127.0.0.1:8081
```

---

## Check Ubuntu

```bash
cat /etc/os-release
```

---

## Check running code-server

```bash
pgrep -af code-server
```

---

# Quick Start

## Native Termux

```bash
pkg update && pkg upgrade -y

pkg install -y \
  nodejs-lts \
  python \
  make \
  clang \
  git

npm install -g code-server

code-server \
  --bind-addr 127.0.0.1:8080
```

Open:

```text
http://127.0.0.1:8080
```

---

## Ubuntu + PRoot

```bash
pkg update && pkg upgrade -y

pkg install -y \
  proot \
  proot-distro \
  curl \
  git

proot-distro install ubuntu

proot-distro login ubuntu
```

Inside Ubuntu:

```bash
apt update

apt install -y \
  curl \
  git \
  build-essential \
  python3

curl -fsSL https://code-server.dev/install.sh | sh

code-server \
  --bind-addr 127.0.0.1:8080
```

Open:

```text
http://127.0.0.1:8080
```

---

# Which Method Should You Use?

### Choose Native Termux if:

* You want the simplest setup.
* You mainly need code-server, Git, Node.js, and Python.
* You want fewer layers between Android and your development environment.

### Choose Ubuntu + PRoot if:

* You need a more traditional Linux userspace.
* You need Ubuntu/Debian packages.
* You encounter compatibility problems with native Termux packages.
* You plan to use more Linux-oriented development tooling.

**Recommended:** Start with **Native Termux**. If you hit compatibility limitations, move to **Ubuntu + PRoot**.

---

<p align="center">
  <sub>Android • Termux • Ubuntu • code-server</sub>
</p>
```
