# 🛡️ The $15 Ad-Blocking Puck (Pi-hole on Pi Zero 2 W)

> **Total Cost:** ~$15  
> **Time to Build:** ~30 mins (or 3 hours if things break!)  
> **Goal:** Network-wide ad blocking and malware protection.

## 📖 Introduction
I spent my weekend battling intrusive ads and malware. I get tired of the constant barrage of pop-ups and unwanted ads following me everywhere online. More importantly, I wanted to ensure my network wouldn't fall victim to deceptive links that could introduce malware into my system. Therefore I decided to built a dedicated DNS Server (Pi-hole) that "scrubs" the internet for my entire house.

This repository documents the exact steps to replicate this setup using a **headless (wireless)** configuration.

---

## 🛠️ Hardware Requirements
* **Raspberry Pi Zero 2 W** (~$15)
* **MicroSD Card** (8GB or larger, Class 10 recommended)
* **Power Supply** (Micro-USB)

---

## 🚀 Setup Guide

### Part 1: The Headless Setup (Critical Step)
Since the Pi Zero 2 W is wireless, we configure Wi-Fi *before* the first boot using the **Raspberry Pi Imager**.

<img width="680" height="482" alt="pi-hole9" src="https://github.com/user-attachments/assets/22a572d3-490a-4680-b22d-50369e472cf7" />

1.  Download and install the [Raspberry Pi Imager](https://www.raspberrypi.com/software/).
2.  **Choose OS:** Raspberry Pi OS (64-bit).
3.  **Choose Storage:** Select your MicroSD card.
4.  **⚙️ Click the Settings (Gear Icon):**
    * [x] **Set Hostname:** `pihole`
    * [x] **Enable SSH:** Select "Use password authentication".
    * [x] **Set Username/Password:** (e.g., `admin` / `yourpassword`)
    * [x] **Configure Wireless LAN:** Enter your Home SSID and Password.
5.  **Write** to the SD card.



### Part 2: First Boot & Connection

<img width="680" height="600" alt="pi-hole10" src="https://github.com/user-attachments/assets/43351976-c1d2-4b68-956e-2985066c54a5" />

1.  Insert the SD card and plug in the power.
2.  Wait **~3 minutes** for the Pi to boot and connect to Wi-Fi.
3.  Open your terminal and find the IP address:
    ```bash
    ping pihole
    ```
4.  SSH into the device:
    ```bash
    ssh admin@<IP_ADDRESS>
    ```
5.  Update the system:
    ```bash
    sudo apt update && sudo apt full-upgrade -y
    ```
<img width="861" height="485" alt="piholedns11" src="https://github.com/user-attachments/assets/0af88e5b-d9f8-4339-8e6a-a10bc4d9e864" />

### Part 3: Installation & Configuration
1.  **Set a Static IP:** It is highly recommended to reserve a static IP for the Pi in your Router's DHCP settings so the address doesn't change.
2.  **Run the Pi-hole Installer:**
    ```bash
    curl -sSL [https://install.pi-hole.net](https://install.pi-hole.net) | bash
    ```
3.  **Follow the Prompts:**
    * **Upstream DNS:** I recommend **Quad9** (filtered, DNSSEC) for added security.
    * **Blocklists:** Select “yes” to add StevenBlack’s Unified Hosts List
    * **Logging:** No (recommended only for troubleshooting).
    * **Privacy mode for FTL:** Select “0 – Show Everything”
    * **After install, set password, from SSH, for Pi-hole with:**
      `sudo pihole setpassword pihole`
    * **Note: you can change `pihole` to a different password**
    * **Shutdown DNS Server:** `sudo shutdown -r now`

### Part 4: Supercharge the Blocklists 🛡️
Standard blocking is good; Pro blocking is better.

<img width="987" height="822" alt="piholedns12" src="https://github.com/user-attachments/assets/4c3a2a83-2cae-41a8-9dd7-1b19f7491f4c" />

1.  Navigate to the admin panel: `http://<IP_ADDRESS>/admin`
2.  Go to **List > Add a new subscribed list**.
3.  Add the **Hagezi Pro** list:
    ```text
    [https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/pro.txt](https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/pro.txt)
    ```
4. Add the **Threat Intelligence** list:
   ```text
    [https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/tif.txt](https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/tif.txt)
    ```
5.  **Crucial Step:** Go to **Tools > Update Gravity** to make the list active.

---

## 📡 Final Step: Router Configuration
To make this work for every device in your house (Phones, Smart TVs, Laptops):

1.  Log into your Router's admin page.
2.  Find **DHCP** or **LAN Settings**.
3.  Change the **Primary DNS Server** to the IP address of your Raspberry Pi.
4.  Save and Reboot router.

---

## ✅ Results
* **Cleaner Browsing:** No more intrusive pop-ups.
* **Malware Protection:** Known malicious domains are blocked at the DNS level.
* **Network Speed:** Less bandwidth wasted on loading ads.

---

### 📝 Notes
* If `ping pihole` doesn't work, check your router's "Connected Devices" list to find the IP address manually.
* Always change the default password after installation.
