# OpenVPN Access Server Docker Setup

This repository contains a **Docker Compose setup** for running **OpenVPN Access Server** on a VPS or local machine with **NGINX reverse proxy**. The setup allows easy deployment and access to both the **Admin UI** and **Client UI**.

## Features

* OpenVPN Access Server in Docker
* Persistent configuration using Docker volumes
* Admin UI accessible via NGINX reverse proxy
* Client UI accessible via NGINX reverse proxy
* Easy configuration for local testing or VPS deployment
* Compatible with personal domain names

## Prerequisites

* Docker & Docker Compose installed
* NGINX installed for reverse proxy
* VPS or local machine with public IP
* Domain name (optional, for reverse proxy)

## NGINX Reverse Proxy Setup

Create an NGINX config for your domain or IP:

```bash
sudo nano /etc/nginx/sites-available/vpn.example.com.conf
```
paste the below config

```nginx
server {
    listen 80;
    server_name vpn.example.com;

    # Redirect HTTP to HTTPS (if using SSL, optional)
    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443;
    server_name vpn.example.com;

    # Admin UI (optional path /admin)
    location /admin/ {
        proxy_pass http://127.0.0.1:943/admin/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Client UI
    location / {
        proxy_pass http://127.0.0.1:9443/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```
Enable and Restart
```bash
sudo ln -s /etc/nginx/sites-available/vpn.example.com.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Notes**:

* OpenVPN client port (`1194/udp`) cannot be proxied through NGINX; keep it open in firewall.
* Admin UI accessible at `/admin/`
* Client UI accessible at `/`


## Initial Setup

1. Start the Docker container:

```bash
docker compose up -d
```

2. Check logs to get the default admin password (if not set via environment variable):

```bash
docker logs openvpn-as
```

3. Access the Admin UI:

```
http://<your-domain-or-ip>/admin
```

4. Access the Client UI:

```
http://<your-domain-or-ip>/
```

## Add Users

1. Log in to Admin UI.
2. Go to **User Management → User Permissions**.
3. Add a user and set a password.
4. User logs in to Client UI and downloads `.ovpn` profile.

## Connect with OpenVPN Client

1. Install official OpenVPN client (Windows, macOS, Linux, iOS, Android).
2. Import `.ovpn` profile from Client UI.
3. Connect using username and password.

## Firewall / Port Requirements

* `1194/udp` → OpenVPN client connections
* `943/tcp` → Admin UI (proxied via NGINX)
* `9443/tcp` → Client UI (proxied via NGINX)
* `80/tcp` → NGINX HTTP redirect (if using SSL in the future)
* `443/tcp` → NGINX HTTPS (if using SSL in the future)

## Notes

* SSL (Let’s Encrypt) can be added later if desired.
* Running locally is fine for testing, but public VPS recommended for remote connections.
* Docker volume `openvpn_data` persists all server configs, users, and certificates.