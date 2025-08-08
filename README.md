# Odoo ERP Infrastructure & Custom Addons Repository

## 📌 Introduction

This repository provides everything needed to deploy and customize an **Odoo ERP** application in a production environment.

It includes:

- **Nginx configuration files** tailored for Odoo’s web and longpolling services.
- **Custom-built addons** for specific business needs.
- **Open-source addons** sourced from the community to enhance functionality.

This setup helps ensure your Odoo environment is secure, performant, and extensible.

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

The `infrastructure/nginx/` directory contains the **reverse proxy configuration** for Odoo. This configuration is designed to:

- Serve Odoo via **HTTP/HTTPS** with SSL/TLS security.
- Separate **longpolling traffic** for improved real-time performance.
- Optimize performance with caching and compression.
- Keep logs organized for easier monitoring and troubleshooting.

### Example Configuration: `odoo.conf`

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

### Key Points:

- **Reverse Proxy:** Directs incoming requests to the Odoo backend.
- **SSL Support:** Add `listen 443 ssl;` and include certificate files for HTTPS.
- **Longpolling:** Handles websocket-like requests on port `8072`.
- **Logging:** Maintains separate access and error logs.

---

## ⚙️ Odoo Addons

The `addons/` directory contains modules that extend Odoo’s capabilities. It is divided into two categories:

### 1. `addons/custom/`

For **in-house developed** Odoo modules tailored to your specific business requirements.
Each custom module follows Odoo’s module structure:

- `__manifest__.py` → Module metadata.
- `models/` → Python business logic.
- `views/` → XML UI definitions.
- `security/` → Access control lists and record rules.

### 2. `addons/open_source/`

For **third-party community addons** sourced from the internet or Odoo App Store.
These are integrated into the environment but maintained separately from custom modules.

---

## 📦 Installing Custom & Open-Source Addons

1. Place the addon folder in either:

   - `addons/custom/` for internal modules
   - `addons/open_source/` for community modules

2. Update your Odoo configuration to include both paths:

   ```ini
   addons_path = /path/to/odoo/addons,/path/to/repo/addons/custom,/path/to/repo/addons/open_source
   ```

3. Restart Odoo:

   ```bash
   sudo systemctl restart odoo
   ```

4. Enable Developer Mode in Odoo UI → Install the desired module from **Apps**.

---

## 🚀 Deployment Tips

- Test Nginx configuration before applying:

  ```bash
  sudo nginx -t
  ```

- Reload Nginx after changes:

  ```bash
  sudo systemctl reload nginx
  ```

- Use **Let’s Encrypt** or another CA for SSL certificates.
- Keep `custom` and `open_source` addons separate to avoid upgrade conflicts.

---

## 📄 License

This repository is licensed under the [MIT License](LICENSE).
