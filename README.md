# Web Application Firewall Home Lab using SafeLine WAF (DVWA + Ubuntu + Kali Linux)

## Overview

This project sets up a complete **cybersecurity home lab** on VirtualBox using Ubuntu Server, Kali Linux, DVWA, and SafeLine WAF. The goal is to simulate a vulnerable web application environment, demonstrate attacks like SQL injection, and showcase how a Web Application Firewall (WAF) like **SafeLine** can defend against them.

---

##  Objectives

- Deploy a **vulnerable web app (DVWA)** on an Ubuntu server.
- Perform a **basic SQL injection attack** using Kali Linux.
- **Protect the application** using SafeLine WAF.
- Explore **advanced WAF features** like HTTP flood protection and custom deny rules.

---

##  Prerequisites

- A host machine with at least **4 GB RAM** and **50 GB free disk space**
- **VirtualBox** installed
- Basic Linux command-line knowledge
- Stable internet connection


##  Lab Environment Setup

### 1. Install VirtualBox

* Download from: [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads)
* Install the Extension Pack (optional, but recommended)

### 2. Kali Linux Virtual Machine

* Download from: [Kali Linux Downloads](https://www.kali.org/get-kali)
* VM Settings:

  * Name: `KaliLinux`
  * Type: Linux / Debian (64-bit)
  * RAM: 4 GB minimum
  * Disk: 50 GB dynamically allocated
* Attach the Kali ISO and perform the graphical installation

### 3. Ubuntu Server Virtual Machine

* Download from: [Ubuntu Server LTS](https://ubuntu.com/download/server)
* VM Settings:

  * Name: `UbuntuServer`
  * Type: Linux / Ubuntu (64-bit)
  * RAM: 2 GB minimum
  * Disk: 20 GB dynamically allocated
* Install Ubuntu Server and optionally enable OpenSSH

### 4. Enable Bridged Networking (on both VMs)

* Go to VirtualBox > VM Settings > Network
* Adapter 1: `Bridged Adapter`
* Select your physical NIC (Ethernet or Wi-Fi)

### 5. Install Guest Additions (Optional)

```bash
sudo apt-get update
sudo apt-get install build-essential dkms linux-headers-$(uname -r)
sudo mount /dev/cdrom /media/cdrom
sudo /media/cdrom/VBoxLinuxAdditions.run
```

Restart the VM after installation.

-

## ⚙️ Ubuntu Server Configuration

### 1. System Update & Basic Tools

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y net-tools openssl git
```

### 2. Install the LAMP Stack

```bash
sudo apt-get install -y apache2 php php-mysql mysql-server
```

---

##  Deploy DVWA (Damn Vulnerable Web Application)

### 1. Clone DVWA

```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
sudo chown -R www-data:www-data DVWA
sudo chmod -R 755 DVWA
```

### 2. Configure DVWA MySQL

```bash
sudo mysql -u root -p
CREATE DATABASE dvwa;
CREATE USER 'dvwa_user'@'localhost' IDENTIFIED BY 'p@ssw0rd';
GRANT ALL ON dvwa.* TO 'dvwa_user'@'localhost';
exit
```

Edit DVWA config file at `DVWA/config/config.inc.php` and update DB credentials.

### 3. Change Apache Port to 8080

```bash
sudo nano /etc/apache2/ports.conf
# Change: Listen 80 -> Listen 8080

sudo nano /etc/apache2/sites-available/000-default.conf
# Change: <VirtualHost *:80> -> <VirtualHost *:8080>
# Update DocumentRoot if needed

sudo systemctl restart apache2
```

DVWA should now be accessible at: `http://<Ubuntu-IP>:8080/DVWA`



## 🧭 Local DNS Resolution

Edit `/etc/hosts` on both Ubuntu and Kali:

```bash
sudo nano /etc/hosts
```

Add:

```
<Ubuntu-IP>  webserver.sam
```

Now DVWA is accessible at: `http://webserver.sam:8080/DVWA`



##  Create a Self-Signed SSL Certificate

```bash
sudo openssl genrsa -out /etc/ssl/private/dvwa.key 4096
sudo openssl req -x509 -new -key /etc/ssl/private/dvwa.key -out /etc/ssl/certs/dvwa.crt -days 365
```



##  Install and Configure SafeLine WAF

### 1. Auto Install SafeLine

```bash
bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/manager.sh)" -- --en
```

Note the login credentials and management URL (usually `https://<Ubuntu-IP>:9443`)

### 2. Import SSL Certificate into SafeLine

Use the GUI to import:

* Certificate: `/etc/ssl/certs/dvwa.crt`
* Key: `/etc/ssl/private/dvwa.key`

### 3. Onboard DVWA in SafeLine

* Domain: `webserver.sam`
* Port: `443` (SSL enabled)
* Reverse Proxy: `http://192.168.1.162:8080` (adjust IP)
* Attach imported SSL cert
* Save and test: `https://webserver.sam`



##  Simulate SQL Injection Attack

### 1. From Kali Linux:

* Visit: `https://webserver.sam`
* Login with default: `admin / password`
* Set security level to `low`
* Go to **SQL Injection** section
* Try payloads like:

```sql
admin' OR '1'='1
```

SafeLine WAF should detect and block the attack depending on settings.



##  SafeLine Advanced Configurations

### 1. HTTP Flood Protection

* Enable under WAF > Security Settings
* Set rate limits (e.g., 10 requests/sec)
* Use `ab`, `siege`, or `burp intruder` from Kali to simulate flooding

### 2. Auth Gateway

* Enable Auth > Set credentials
* Try accessing DVWA; SafeLine will prompt for login before proxying

### 3. Custom Deny Rule

* From WAF console, add:

```
Source IP: 192.168.1.50 (your Kali IP)
Action: Deny
```

* Attempt access from Kali; you should get blocked


##  Final Notes

*  **Troubleshoot** by checking Apache logs, SafeLine logs, and Kali terminal
*  **Keep this lab isolated** from your production or main networks
*  **Regularly update** all components for security patches



##  Acknowledgments

* Special thanks [The Social Dork on YouTube](https://www.youtube.com/@thesocialdork1133) for providing inspiring and insightful cybersecurity lab content. This project was made much easier by following their techniques and walkthroughs.


##  To-Do (Optional Enhancements)

* Add Splunk or ELK for logging
* Add multi-stage attack chain testing (LFI > RCE)
* Integrate with ModSecurity or CrowdSec
* Set up Suricata for network-level detection
