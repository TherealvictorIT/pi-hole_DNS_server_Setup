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
Since the Pi Zero 2 W is wireless, we need to configure the Wi-Fi *before* the first boot using the **Raspberry Pi Imager**.

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
The standard blocking is good but adding a couple more lists is better. You do not need to do this step but 

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
5.  **Crucial Step:** Go to **Tools > Update Gravity > Update** to make the list active.

---

## 📡 Final Step: Router Configuration
To make this work for every device in your house (Phones, Smart TVs, Laptops):

1.  Log into your Router's admin page.
2.  Find **DHCP** or **LAN Settings**.
3.  Change the **Primary DNS Server** to the IP address of your Raspberry Pi.
4.  (Optional) Set DNS Server 2 to `8.8.8.8` (Google) as a backup, OR to your Proxmox server IP if you set up a second one there.
5.  Save and Reboot router.

---

## ✅ Results
* **Cleaner Browsing:** No more intrusive pop-ups.
* **Malware Protection:** Known malicious domains are blocked at the DNS level.
* **Network Speed:** Less bandwidth wasted on loading ads.

---

### 📝 Notes
* If `ping pihole` doesn't work, check your router's "Connected Devices" list to find the IP address manually.
* Always change the default password after installation.

# Pi-Hole Installation Part 2 | 🕵️‍♂️ Making Pi-hole Invisible with Unbound

> **Goal:** Stop relying on Google/Cloudflare/ISP DNS. Become your own Recursive DNS Server.  
> **Privacy Level:** Maximum.  
> **Difficulty:** Intermediate (Command Line).

## 🤔 Why Unbound?
Standard Pi-hole blocks ads, but it still forwards your DNS requests to an "upstream" provider (like Google `8.8.8.8`). This means:
1.  They know your browsing history.
2.  They can censor results.
3.  If they go down, you go down.

**Unbound** changes this. It talks directly to the global Root Servers. No middleman. No logs. True privacy.

---

## 🚀 Installation Guide

### Step 1: Install Unbound

<img width="832" height="119" alt="unbound" src="https://github.com/user-attachments/assets/51ab86e9-1084-48d2-9828-d24d7c4c0fff" />

SSH into your Pi-hole and run the installer:

```bash
sudo apt install -y unbound
```

### Step 2: Get the current DNS root hints:
* The list of primary root servers which are serving the domain
```bash
sudo curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
```
### Step 3: Minimal privacy-focused Unbound config:
* You will access the file and copy the text bellow
```bash
sudo vim /etc/unbound/unbound.conf.d/pi-hole.conf
```
[Copy/Paste]
```bash
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to no if you don't have IPv6 connectivity
    do-ip6: yes

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes
		
		 # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10

    # Ensure no reverse queries to non-public IP ranges (RFC6303 4.2)
    private-address: 192.0.2.0/24
    private-address: 198.51.100.0/24
    private-address: 203.0.113.0/24
    private-address: 255.255.255.255/32
    private-address: 2001:db8::/32
```
### Step 4: Enable & restart:
    sudo systemctl enable unbound
    sudo systemctl restart unbound
    sudo systemctl status unbound --no-pager
    
### Step 5: Test Unbound directly on the Pi:

<img width="781" height="374" alt="unbound3" src="https://github.com/user-attachments/assets/1c422fee-6e37-4221-88f6-04e8b9f80d8c" />

```bash
dig @127.0.0.1 -p 5335 google.com +dnssec +multi
```
    
### Step 6: Point Pi-hole to Unbound (as upstream):

<img width="490" height="995" alt="unbound2" src="https://github.com/user-attachments/assets/4171a1a9-f2c9-4bca-af66-6def9bf697fd" />

    1. Log into Pi-hole Web UI > Settings > DNS > Upstream DNS Servers:
    2. Uncheck any public upstreams
    3. Custom 1 (IPv4): 127.0.0.1#5335
    4. Save
    
### Step 7: Test from a client console window:
```bash
nslookup www.facebook.com
```
    
### Step 8: Verify in Pi-hole Web UI:

Check **Query Log** to confirm queries go to **127.0.0.1**

<img width="956" height="115" alt="unbound5" src="https://github.com/user-attachments/assets/ab24b2d7-5a06-4272-b4f0-d1fde31a2e35" />

