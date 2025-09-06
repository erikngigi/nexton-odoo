# Odoo 18 Installation on Ubuntu 24.04

Odoo is an **open-source business management software** written in Python and JavaScript.
It provides a wide range of integrated applications including:

- Customer Relationship Management (CRM)
- E-commerce
- Accounting and Billing
- Manufacturing
- Project Management
- Human Resources and more

This guide walks you through installing **Odoo 18** on **Ubuntu 24.04** with PostgreSQL as its database backend.

---

## 📑 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1. Update the System](#step-1-update-the-system)
3. [Step 2. Install Python and Dependencies](#step-2-install-python-and-dependencies)
4. [Step 3. Install NPM and Node CSS Plugins](#step-3-install-npm-and-node-css-plugins)
5. [Step 4. Install Wkhtmltopdf](#step-4-install-wkhtmltopdf)
6. [Step 5. Install PostgreSQL Database Service](#step-5-install-postgresql-database-service)
7. [Step 6. Create Odoo System and Database User](#step-6-create-odoo-system-and-database-user)
8. [Step 7. Install and Configure Odoo 18](#step-7-install-and-configure-odoo-18)
9. [Step 8. Create Odoo Configuration File](#step-8-create-odoo-configuration-file)
10. [Step 9. Create Odoo Service File](#step-9-create-odoo-service-file)
11. [Step 10. Manage Odoo Service](#step-10-manage-odoo-service)
12. [Customizations](#-customizations)
13. [Directory Structure](#️-directory-structure)
14. [Nginx Configuration for Odoo](#-nginx-configuration-for-odoo)
15. [Odoo Addons](#️-odoo-addons)
16. [Installing Custom & Open-Source Addons](#-installing-custom--open-source-addons)
17. [Deployment Tips](#-deployment-tips)
18. [Congratulations!](#-congratulations)

---

## Prerequisites

- A server running **Ubuntu 24.04**
- User privileges: **root** or a **sudo-enabled non-root user**

---

## Step 1. Update the System

```bash
sudo apt update -y && sudo apt upgrade -y
```

---

## Step 2. Install Python and Dependencies

```bash
sudo apt install build-essential wget git python3-pip python3-dev python3-venv python3-wheel python3 libfreetype6-dev libxml2-dev libzip-dev libsasl2-dev python3-setuptools libjpeg-dev zlib1g-dev libpq-dev libxslt1-dev libldap2-dev libtiff5-dev libopenjp2-7-dev -y
```

Verify installation:

```bash
python3 -V
# Output: Python 3.12.3
```

---

## Step 3. Install NPM and Node CSS Plugins

```bash
sudo apt-get install npm -y
npm install -g less less-plugin-clean-css
sudo apt-get install node-less -y
```

---

## Step 4. Install the Lower Version of Wkhtmltopdf

Odoo works best with **wkhtmltopdf 0.12.5** (instead of the latest versions).
Follow the steps below to install it along with its dependencies.

### 4.1 Download wkhtmltox 0.12.5

```bash
wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb
```

### 4.2 Install wkhtmltopdf and Dependencies

```bash
sudo dpkg -i wkhtmltox_0.12.5-1.bionic_amd64.deb
sudo apt-get install xfonts-75dpi
sudo apt-get update
```

Fix broken dependencies:

```bash
sudo apt --fix-broken install
```

### 4.3 Install LIBSSL1.1 Manually

Since Ubuntu 24.04 does not include `libssl1.1` by default, install it manually:

```bash
sudo wget https://mirror.sit.fraunhofer.de/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1l-1ubuntu1.2_amd64.deb
sudo dpkg -i libssl1.1_1.1.1l-1ubuntu1.2_amd64.deb
```

---

## Step 5. Install PostgreSQL Database Service

```bash
sudo apt-get install postgresql -y
sudo systemctl start postgresql && sudo systemctl enable postgresql
sudo systemctl status postgresql
```

Expected output:

```
Active: active (exited)
```

---

## Step 6. Create Odoo System and Database User

Create Odoo system user:

```bash
sudo useradd -m -U -r -d /opt/odoo18 -s /bin/bash odoo18
```

Create Odoo PostgreSQL user:

```bash
sudo su - postgres -c "createuser -s odoo18"
```

---

## Step 7. Install and Configure Odoo 18

Switch to the Odoo system user:

```bash
su - odoo18
```

Download Odoo:

```bash
git clone https://www.github.com/odoo/odoo --depth 1 --branch 18.0 /opt/odoo18/odoo18
```

Create virtual environment & install requirements:

```bash
python3.12 -m venv odoo18-venv
source odoo18-venv/bin/activate
pip install --upgrade pip
pip3 install wheel
pip3 install -r odoo18/requirements.txt
deactivate
```

Create addons & logs:

```bash
mkdir /opt/odoo18/odoo18-custom-addons
chown -R odoo18:odoo18 /opt/odoo18/odoo18-custom-addons

mkdir -p /var/log/odoo18/ && touch /var/log/odoo18/odoo18.log
chown -R odoo18:odoo18 /var/log/odoo18/
```

---

## Step 8. Create Odoo Configuration File

```bash
sudo nano /etc/odoo18.conf
```

Paste:

```ini
[options]
admin_passwd = StrongPasswordHere
db_host = False
db_port = False
db_user = odoo18
db_password = False
xmlrpc_port = 8069
logfile = /var/log/odoo18/odoo18.log
addons_path = /opt/odoo18/odoo18/addons,/opt/odoo18/odoo18-custom-addons
```

---

## Step 9. Create Odoo Service File

```bash
sudo nano /etc/systemd/system/odoo18.service
```

Paste:

```ini
[Unit]
Description=odoo18

[Service]
Type=simple
SyslogIdentifier=odoo18
PermissionsStartOnly=true
User=odoo18
Group=odoo18
ExecStart=/opt/odoo18/odoo18-venv/bin/python3 /opt/odoo18/odoo18/odoo-bin -c /etc/odoo18.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```

---

## Step 10. Manage Odoo Service

Start and enable Odoo:

```bash
sudo systemctl start odoo18 && sudo systemctl enable odoo18
```

Check status:

```bash
sudo systemctl status odoo18
```

Expected output:

```
Active: active (running)
```

---

## 📌 Customizations

This repository provides everything needed to **deploy and customize an Odoo ERP application** in a production environment.

It includes:

- **Nginx configuration files** tailored for Odoo’s web and longpolling services.
- **Custom-built addons** for specific business needs.
- **Open-source addons** sourced from the community to enhance functionality.

This setup helps ensure your Odoo environment is **secure, performant, and extensible**.

---

## 🛠️ Directory Structure

```
infrastructure/
├── nginx/
│   ├── odoo.conf
│   └── ssl_params.conf
addons/
├── custom/
│   ├── custom_module_1/
│   └── ...
├── open_source/
│   ├── open_module_1/
│   └── ...
```

---

## 🌐 Nginx Configuration for Odoo

The `infrastructure/nginx/` directory contains the **reverse proxy configuration** for Odoo.
This configuration is designed to:

- Serve Odoo via **HTTP/HTTPS with SSL/TLS security**.
- Separate **longpolling traffic** for improved real-time performance.
- Optimize performance with **caching and compression**.
- Keep logs organized for **monitoring and troubleshooting**.

### Example: `odoo.conf`

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://odoo:8069;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /longpolling {
        proxy_pass http://odoo:8072;
    }

    access_log /var/log/nginx/odoo_access.log;
    error_log /var/log/nginx/odoo_error.log;
}
```

**Key Points:**

- **Reverse Proxy:** Directs incoming requests to the Odoo backend.
- **SSL Support:** Add `listen 443 ssl;` and include certificate files for HTTPS.
- **Longpolling:** Handles websocket-like requests on port `8072`.
- **Logging:** Maintains separate access and error logs.

---

## ⚙️ Odoo Addons

The `addons/` directory contains modules that extend Odoo’s capabilities:

1. **`addons/custom/`**

   - In-house developed Odoo modules tailored for your business.
   - Example structure:

     ```
     __manifest__.py   → Module metadata
     models/           → Python business logic
     views/            → XML UI definitions
     security/         → Access control lists & record rules
     ```

2. **`addons/open_source/`**

   - Third-party community addons from the Odoo App Store or GitHub.
   - Maintained separately from custom modules to prevent conflicts.

---

## 📦 Installing Custom & Open-Source Addons

1. Place addons in the appropriate directory:

   - `addons/custom/` → Internal modules
   - `addons/open_source/` → Community modules

2. Update Odoo config to include both paths:

```ini
addons_path = /path/to/odoo/addons,/path/to/repo/addons/custom,/path/to/repo/addons/open_source
```

3. Restart Odoo:

```bash
sudo systemctl restart odoo18
```

4. Enable **Developer Mode** in Odoo → Install module from **Apps**.

---

## 🚀 Deployment Tips

- Test Nginx configuration:

```bash
sudo nginx -t
```

- Reload Nginx after changes:

```bash
sudo systemctl reload nginx
```

- Use **Let’s Encrypt** or another CA for SSL certificates.
- Keep **custom** and **open_source** addons separate to avoid upgrade conflicts.
