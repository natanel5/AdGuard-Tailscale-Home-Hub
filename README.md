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

* **Hardware:** Raspberry Pi (3B+, 4, or 5 recommended) with 1GB+ RAM.
* **Storage:** 16GB+ MicroSD card.
* **OS:** Raspberry Pi OS Lite (Debian 12) 64-bit.
* **Network:** A static IP assigned to your Pi.

## 💻 Installation

### 🛠️ Step 1: Operating System Setup
To ensure a lightweight and stable environment, we use **Raspberry Pi OS Lite (64-bit)**. This version lacks a desktop interface, which saves system resources for our Docker services.

#### 1. Download Raspberry Pi Imager
First, download and install the official imaging tool for your operating system:
**Official Website:** [raspberrypi.com/software](https://www.raspberrypi.com/software/)

#### 2. Prepare the MicroSD Card
* Connect your MicroSD card to your computer.
> [!CAUTION]
> **Warning:** Ensure the card is empty or backed up, as the flashing process will **permanently erase** all existing data on the drive.

#### 3. Configure the Imager
Open the Raspberry Pi Imager and follow these steps:

**Device:** Select your model (e.g., Raspberry Pi 5)<br>
**Operating System:** Choose Raspberry Pi OS Lite (64-bit)<br>
**Storage:** Select your MicroSD card from the list<br>
**Customization:**
* **Hostname:** Set a unique name (e.g., `adguard-hub`). This allows you to connect via `<hostname>.local`.
* **User:** Create your primary admin username and password.
* **Wireless LAN:** Configure your WiFi credentials if you are not using a LAN (Ethernet) cable.
* **Localisation:** Set your time zone and keyboard layout (e.g., `Asia/Jerusalem`).
* **Remote Access:** You **MUST** check **Enable SSH** and select **Allow password authentication**.
* **Raspberry Pi Connect:** Ensure this is set to **OFF** (we will manage the server via SSH and Tailscale).
  
> [!IMPORTANT]
> **SSH must be enabled.** This is the only way to access the Pi remotely for the subsequent installation steps.

#### 4. Flashing and Initial Boot
Once you have saved your customization settings, click **YES** to start the flashing process.<br>
Wait for the flashing process to finish and for the "Write Successful" confirmation before removing the card. <br>
Insert the MicroSD card into your Raspberry Pi and connect the power supply to begin the initial boot.

###  🐳 Step 2: Docker Infrastructure
(Next section to be added: Docker installation and Docker-Compose setup)
