AdGuard & Tailscale Home Hub 🛡️🚀
A professional, Docker-based home network stack designed for privacy, security, and global accessibility. This project transforms your Raspberry Pi into a centralized DNS filtering engine with an encrypted private backbone.

🌟 Overview
This repository provides a production-ready deployment of AdGuard Home, Unbound, and Tailscale. By combining these tools, you achieve network-wide ad-blocking, recursive DNS privacy (no more ISP tracking), and a secure "Exit Node" VPN to access your home network and Israeli IP address from anywhere in the world.

Key Features
Network-Wide Filtering: Block ads, trackers, and malware on every device in your home.

Recursive DNS: Use Unbound to resolve queries directly from Root Servers for ultimate privacy.

Global VPN (Exit Node): Securely browse the web as if you are sitting in your living room, even when traveling abroad.

Service Control: Easily block or limit access to social media (TikTok, etc.) and other services via a clean UI.

🏗️ Architecture
The traffic flows as follows:

Client requests a domain (e.g., google.com).

AdGuard Home checks if it should be blocked.

If allowed, Unbound resolves the IP recursively (bypassing ISP DNS).

Tailscale provides an encrypted tunnel for remote devices to use this stack securely.

📋 Prerequisites
Before you begin, ensure you have the following:

Hardware: Raspberry Pi (3B+, 4, or 5 recommended) with 1GB+ RAM.

Storage: 16GB+ MicroSD card.

OS: Raspberry Pi OS Lite (Debian 12) 64-bit.

Network: A static IP assigned to your Pi.

🛠️ Step 1: Operating System Setup

1. Flash the OS
   Use the Raspberry Pi Imager to flash Raspberry Pi OS Lite (64-bit).

Note: Click on "Edit Settings" in the Imager to:

Set a Hostname (e.g., AdGuard-Hub).

Enable SSH (using password or public key).

Configure your WiFi credentials.

2. Initial Connection
   Once flashed, insert the SD card into your Pi and power it on. Connect via SSH from your terminal:

Bash
ssh <username>@<hostname>.local 3. System Update
Always start with a fresh update:

Bash
sudo apt update && sudo apt upgrade -y
🐳 Step 2: Docker Infrastructure
(Next section to be added: Docker installation and Docker-Compose setup)
