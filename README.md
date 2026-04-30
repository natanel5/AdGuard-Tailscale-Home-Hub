# 🚀 AdGuard & Tailscale Home Hub 🛡️

**Ad blocker, private DNS server, and secure remote access gateway.** This project turns your Raspberry Pi into a central hub that cleans your internet from ads and lets you connect back home securely from any device, anywhere.

## 🌟 Overview

I built this project to solve three main problems that standard home routers can't handle:

**Network-Wide Ad Blocking:** Instead of installing ad-blockers on every phone, laptop, and other devices, AdGuard Home handles it at the source. If a device is on your WiFi, the ads are gone.

**Privacy & Tracking:** By default, your ISP (Internet Service Provider) can see every website you visit through their DNS. This stack uses Unbound to bypass them and talk directly to the source.

**Secure Remote Access:** When traveling, you often need a secure connection or your home's IP address to access local services. Tailscale creates an encrypted tunnel back to your Pi, acting as your own personal VPN (Exit Node).

### Key Features

**Network-Wide Filtering:** Full control to block ads, apps, websites, trackers, and malware on every device in your home network.

**Recursive DNS:** Use Unbound to resolve queries directly from Root Servers for ultimate privacy.

**Global VPN (Exit Node):** Securely browse the web as if you are sitting in your living room, even when traveling abroad.

**Service Control:** Easily block or limit access to social media (TikTok, Instagram, etc.) and other services via a clean UI.

## 🏗️ Architecture

The traffic flows as follows:

```mermaid
graph LR;
    A(["Client requests a domain (website/app)."]) --> B(["AdGuard Home checks if <br/>it should be blocked."]);
    B --> C(["If allowed,<br/> Unbound resolves the <br/>IP recursively"]);
    C --> A;
```

> **Note:** Tailscale provides an encrypted tunnel for remote devices to connect to the home network and use this stack securely from anywhere.

## 📋 Prerequisites

Before you begin, ensure you have the following:

- **Hardware:** Raspberry Pi (3B+, 4, or 5 recommended) with 1GB+ RAM.
- **Storage:** 16GB+ MicroSD card.
- **OS:** Raspberry Pi OS Lite (Debian 12) 64-bit.
- **Network:** A static IP assigned to your Pi.

## 💻 Installation

### 🛠️ Step 1: Operating System Setup

To ensure a lightweight and stable environment, we use **Raspberry Pi OS Lite (64-bit)**. This version lacks a desktop interface, which saves system resources for our Docker services.

#### 1. Download Raspberry Pi Imager

First, download and install the official imaging tool for your operating system:
**Official Website:** [raspberrypi.com/software](https://www.raspberrypi.com/software/)

#### 2. Prepare the MicroSD Card

- Connect your MicroSD card to your computer.
  > [!CAUTION]
  > **Warning:** Ensure the card is empty or backed up, as the flashing process will **permanently erase** all existing data on the drive.

#### 3. Configure the Imager

Open the Raspberry Pi Imager and follow these steps:

**Device:** Select your model (e.g., Raspberry Pi 5)<br>
**Operating System:** Choose Raspberry Pi OS Lite (64-bit)<br>
**Storage:** Select your MicroSD card from the list<br>
**Customization:**

- **Hostname:** Set a unique name (e.g., `adguard-hub`). This allows you to connect via `<hostname>.local`.
- **User:** Create your primary admin username and password.
- **Wireless LAN:** Configure your WiFi credentials if you are not using a LAN (Ethernet) cable.
- **Localisation:** Set your time zone and keyboard layout (e.g., `Asia/Jerusalem`).
- **Remote Access:** You **MUST** check **Enable SSH** and select **Allow password authentication**.
- **Raspberry Pi Connect:** Ensure this is set to **OFF** (we will manage the server via SSH and Tailscale).

> [!IMPORTANT]
> **SSH must be enabled.** This is the only way to access the Pi remotely for the subsequent installation steps.

#### 4. Flashing and Initial Boot

Once you have saved your customization settings, click **YES** to start the flashing process.<br>
Wait for the flashing process to finish and for the "Write Successful" confirmation before removing the card. <br>
Insert the MicroSD card into your Raspberry Pi and connect the power supply to begin the initial boot.

### 🐳 Step 2: Docker Infrastructure

To run our services in an isolated environment, we need to install Docker and Docker Compose.

#### 1. Update the package list and upgrade all installed packages to their latest versions:

```bash
sudo apt update && sudo apt upgrade -y
```

#### 2. Download and run the official Docker installation script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

#### 3. Add your current user to the Docker group to run containers without using sudo:

```bash
sudo usermod -aG docker $USER
```

#### 4. Confirm that Docker and Docker Compose are installed and working correctly:

```bash
docker --version && docker compose version
```

## 🚀 Step 3: Deploying the Hub

Now that Docker is ready, we will pull the project files from GitHub to the Raspberry Pi.

### 1. Clone the Repository

Navigate to your home directory and clone this project:

```bash
cd ~ && git clone https://github.com/natanel5/AdGuard-Tailscale-Home-Hub.git
```

Move into the newly created folder to start managing the services:

```bash
cd AdGuard-Tailscale-Home-Hub
```

Start the services in detached mode (running in the background):

```bash
sudo docker-compose up -d
```

### 🛡️ Step 4: Configuring the AdGuard

We are going to set up our first service, **AdGuard Home!**

#### Initial settings

First, open a browser on your computer and navigate to:

```Plaintext
http://<your-pi-ip>:3000
```

> [!NOTE]
> **Note:** Port 3000 is only used for the initial configuration.<br>
> Once set up, the main dashboard will be accessible via the standard port 80.

Now we can see the AdGuard Home setup!

Click the **Get Started** button and leave the settings as follows:

- **Admin Web Interface**: All interfaces on port 80

- **DNS Server**: All interfaces on port 53

In the next step, choose a username and a password.

> [!IMPORTANT]
> **Important:** Remember these login credentials, as this is your management dashboard.

On the next page, you'll see a guide on connecting your AdGuard to the router;<br>
You can **ignore** it and click **Next** because we'll cover it later.

#### Adding blocklists:

Go to **Filters -> DNS Blocklists**<br>
AdGuard includes default lists; ensure they are enabled.

You can add more by selecting **Add blocklist -> Choose from the list.**
I recommend enabling the default lists plus your specific **Regional filters**.<br>
But it's important to do your **own research**.

> [!TIP]
> **Optional:** You can go to **Filters -> Blocked services** for blocking specific apps, websites, or
> online programs from your network.

#### Connecting AdGuard to your Network:

To add AdGuard to your home network, we have 3 steps:

#### Enter the router management UI:

Navigate to your router's address in the browser.

Usually it's **192.168.1.1**, but sometimes it can be **192.168.0.1** or **10.0.0.1**.

If it's none of the above, you can look at the Network settings on any device connected to the same network and look for the **Default Gateway**; this is your router's IP address.

#### Assigning a Static IP to your Pi:

Your router's DHCP server assigns IP addresses dynamically.
Sometimes, if your Pi or your router restarts, the DHCP server might assign a new IP to the Pi.
This would break your network configuration, as the router would still try to forward DNS requests to the old (non-existent) address.

On your router management UI, find a setting called **Static leases** or **Address Reservation** (every UI has different names) and assign a static IP to your Pi based on its **MAC address**. <br>

To find your MAC address on the Pi terminal, run this command:

```bash
cat /sys/class/net/$(ip route show default | awk '/default/ {print $5}')/address
```

The output should follow this format: **00:1a:2b:3c:4d:5e**

> [!IMPORTANT]
> The MAC address depends on your connection type (Wi-Fi or LAN).<br>
> If you switch between them, you will need to update your DHCP reservation with the new MAC address.

#### Forward all your DNS queries to the Pi:

For the final part, we need to make sure all the DNS queries route through your AdGuard, located inside your home Pi.

We have 2 choices:

- **Option 1 - Router-Level Setup (Recommended):**<br>
  Find the setting called **DNS Server Address** (typically found under **LAN** or **Internet** settings) and change the default DNS to your Pi's static IP.

- **Option 2 - Device-Level Setup:**<br>
  You can go to the **AdGuard management UI** and navigate to the **setup guide** tab, and see the
  instructions on any device.
