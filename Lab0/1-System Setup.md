# Install WSL2 + ROS 2 Humble and Verify It Works (Minimal)

Goal: **Windows → WSL2 → Ubuntu 22.04 → ROS 2 Humble → quick test**

https://mailaub-my.sharepoint.com/:u:/g/personal/sb137_aub_edu_lb/IQCuGZtIeEwyTrmKv76ediiVAdvtMH73BHDwPcQjRdvohNA?e=YQ8xuk
---

## 1) Install WSL2 + Ubuntu 22.04

### 1.1 Open Command Prompt as Administrator

Start → type `cmd` → right-click **Command Prompt** → **Run as administrator**

### 1.2 Install WSL + Ubuntu 22.04

```bat
wsl --install -d Ubuntu-22.04
```

Restart Windows if prompted.

### 1.3 Open Ubuntu

From Command Prompt:

```bat
wsl -d Ubuntu-22.04
```

Or open **Ubuntu 22.04 LTS** from the Start menu.

The first time Ubuntu starts, create your:

* Linux username
* Linux password

> When typing the Linux password, nothing appears on screen. This is normal.

---

## 2) Verify Ubuntu and WSL2

### Check Ubuntu version

Inside Ubuntu:

```bash
lsb_release -a
```

You should see:

```text
Ubuntu 22.04
Codename: jammy
```

### Check WSL version

From Windows Command Prompt:

```bat
wsl -l -v
```

You should see something similar to:

```text
NAME            STATE      VERSION
Ubuntu-22.04    Running    2
```

Make sure **VERSION = 2**.

---

## 3) Update Ubuntu

Inside Ubuntu:

```bash
sudo apt update
sudo apt upgrade -y
```

Then install the required tools:

```bash
sudo apt install -y curl software-properties-common
```

---

## 4) Install ROS 2 Humble

### 4.1 Enable the Ubuntu Universe repository

```bash
sudo add-apt-repository universe
sudo apt update
```

### 4.2 Add the ROS 2 repository

```bash
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F'"' '{print $4}')
```

Then:

```bash
curl -L -o /tmp/ros2-apt-source.deb \
"https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
```

Install it:

```bash
sudo dpkg -i /tmp/ros2-apt-source.deb
```

---

## 5) Install ROS 2 Humble Desktop

```bash
sudo apt update
sudo apt install -y ros-humble-desktop
```

Optional development tools:

```bash
sudo apt install -y ros-dev-tools
```

---

## 6) Automatically Load ROS 2

Add ROS 2 to `.bashrc`:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

Reload the terminal configuration:

```bash
source ~/.bashrc
```

Now ROS 2 will automatically be available whenever you open Ubuntu.

---

# 7) Test ROS 2 — Required

## 7.1 Check the ROS 2 CLI

```bash
ros2 --help
```

If ROS 2 is installed correctly, the ROS 2 help menu should appear.

You can also check the ROS distribution:

```bash
echo $ROS_DISTRO
```

Expected output:

```text
humble
```

---

## 7.2 Talker / Listener Test

Open **two Ubuntu terminals**.

### Terminal 1 — Talker

```bash
ros2 run demo_nodes_cpp talker
```

You should see messages such as:

```text
Publishing: 'Hello World: 1'
Publishing: 'Hello World: 2'
Publishing: 'Hello World: 3'
```

### Terminal 2 — Listener

```bash
ros2 run demo_nodes_cpp listener
```

You should see:

```text
I heard: [Hello World: 1]
I heard: [Hello World: 2]
I heard: [Hello World: 3]
```

✅ If the listener receives the talker's messages, **ROS 2 Humble is working correctly**.

---

# Final Check

Your setup should now be:

```text
Windows
   ↓
WSL2
   ↓
Ubuntu 22.04 LTS (Jammy)
   ↓
ROS 2 Humble
   ↓
Talker ↔ Listener
```

✅ Setup complete.
