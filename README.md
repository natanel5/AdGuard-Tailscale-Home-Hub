# 🚀 AdGuard & Tailscale Home Hub 🛡️

**Ad blocker, private DNS server, and secure remote access gateway.** This project turns your Raspberry Pi into a central hub that cleans your internet from ads and lets you connect back home securely from any device, anywhere.

## 🌟 Overview

I built this project to solve three main problems that standard home routers can't handle:

**Network-Wide Ad Blocking:** Instead of installing ad-blockers on every phone, laptop, and other devices, AdGuard Home handles it at the source. If a device is on your WiFi, the ads are gone.

**Privacy & Tracking:** By default, your ISP (Internet Service Provider) can see every website you visit through their DNS. This stack uses Unbound to bypass them and talk directly to the source.

**Secure Remote Access:** When traveling, you often need a secure connection or your home's IP address to access local services. Tailscale creates an encrypted tunnel back to your Pi, acting as your own personal VPN (Exit Node).

### Key Features

**Network-Wide Filtering:** Full control to block ads, apps, websites, trackers, and malware on every device in your home network. <br>
**Recursive DNS:** Use Unbound to resolve queries directly from Root Servers for ultimate privacy.<br>
**Global VPN (Exit Node):** Securely browse the web as if you are sitting in your living room, even when traveling abroad.<br>
**Service Control:** Easily block or limit access to social media (TikTok, Instagram, etc.) and other services via a clean UI.

## 🏗️ Architecture

The traffic flows as follows:

```mermaid
graph LR;
    A(["Client requests a domain (website/app)."]) --> B(["AdGuard Home checks if <br/>it should be blocked."]);
    B --> C(["If allowed,<br/> Unbound resolves the <br/>IP recursively"]);
    C --> A;
```

> [!NOTE]
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

<br>

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

<br>

### 🚀 Step 3: Deploying the Hub

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

<br>

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

#### Additional configurations (Not mandatory):

Go to **Settings -> DNS settings -> DNS Cache Configuration** and adjust the following configurations:

- **Cache size** - Change the number to **67,108,864** (64MB);<br>
  Increasing the cache size reduces recursive lookups and improves overall network latency.

- **Optimistic caching** - Enable this to improve performance;<br>
  This allows AdGuard to serve expired entries from the cache while simultaneously updating them in the background.

**Custom Filtering Rules:**<br>

Lastly go to **Filters -> Custom filtering rules** and enter these rules. This is where you can manage your own whitelist or blacklist.

```Plaintext
! Samsung News (Unblock Taboola for Samsung News Feed)
@@||cdn.taboola.com^$important
@@||api.taboola.com^$important
@@||images.taboola.com^$important

! Meta / Instagram (Media & CDN)
@@||fbcdn.net^$important

! Meta General Domains
@@||facebook.net^$important
@@||instagram.net^$important
```

This is the place you can enter your **own** filtering rules.

> [!NOTE]
> This is a curated list based on my personal experience with AdGuard Home, and I will update it periodically.

### 🛡️ Step 4: Configuring the Unbound service

Now we can set our second service, **The Unbound service!**
This service will help us to resolve queries privately and securely.

The Unbound service can resolve queries with the official **ICANN** DNS servers. That can give us more privacy as our queries don't go through third parties like your ISP or google.

To get the ICANN DNS server information we run this command that takes the info from the official site and restart the service:

```bash
wget https://www.internic.net/domain/named.root -qO /home/pi_server/AdGuard-Tailscale-Home-Hub/unbound/root.hints && docker restart unbound
```

Next we make a crontab that every 6 months refresh the root.hints file and restart unbound.

```bash
crontab -e
```

Choose the /bin/nano and paste the following code at the end of the file:

```bash
0 0 1 */6 * wget https://www.internic.net/domain/named.root -qO /home/pi_server/AdGuard-Tailscale-Home-Hub/unbound/root.hints && docker restart unbound
```

Sets up the master security key so Unbound can prove that website addresses are real and haven't been faked

```bash
docker run --rm -v $(pwd)/unbound:/etc/unbound --entrypoint unbound-anchor klutchell/unbound:latest -a /etc/unbound/root.key
```

To allow the Unbound service to write the root.key file we give read & write permission to the root.key file and full permissions to the unbound directory.

```bash
sudo chmod 666 ./unbound/root.key
```

```bash
sudo chmod 777 ./unbound
```

Now the server is up and you can check it with the dig tool.

Update and download the tool

```bash
sudo apt update && sudo apt install dnsutils -y
```

Now you can check if it's working.

```bash
dig google.com @127.0.0.1 -p 5335
```

If you get **status: NOERROR** you are good to go.

Now the next step is to direct the AdGuard to use the Unbound service.

Go to the AdGuard configuration page and go to **Settings -> DNS settings -> Upstream DNS servers** and erase all the default dns servers and put only **127.0.0.1:5335** so we are redirecting every DNS query to our Unbound service.

Next we will fail-proofing our system so if the unbound service is down we redirect the queries to more reliable servers like Google, Cloudflare, Quad9 etc.

On the same block go to **Fallback DNS servers** and write one of the following options.

- 1.1.1.1 - Cloudflare DNS for fast, reliable, secure and private.
- 8.8.8.8 - Google DNS for fast, reliable and secure.
- 9.9.9.9 - Quad9 DNS for fast, reliable, secure and even more private.
- any other DNS server you want.

Last go to **Upstream timeout** setting that allow you to choose how much time it takes for the AdGuard to redirect the queries to the fallback DNS servers. The default is 10 seconds, but my recommendation is 3-5 seconds and hit apply.

> [!NOTE]
> You can see that everything works correctly if you press the **Test upstreams** button and get the **Specified DNS servers are working correctly** message.
