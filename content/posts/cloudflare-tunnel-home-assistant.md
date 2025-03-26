+++
date = '2025-03-26T14:21:35-05:00'
draft = false
title = 'External access for Home Assistant using Cloudflare Tunnels'
+++

# 🌐 Set Up External Access for Home Assistant Using the Cloudflare Tunnel Add-on

This guide walks you through exposing your Home Assistant instance securely over the internet using **Cloudflare Tunnel** and the **Cloudflared add-on** with a **token-based config**.

---

## 🧱 Requirements

- Home Assistant OS (e.g. on Raspberry Pi) running on port 8123.
- Domain name (e.g. `your-custom-domain.com`)
- Cloudflare account managing the domain
- Cloudflared Home Assistant add-on installed

---

## 🔐 Step 1: Set Up Domain in Cloudflare

1. Log into [https://dash.cloudflare.com](https://dash.cloudflare.com)
2. Add your domain if not already added (e.g. `your-custom-domain.com`)
3. Follow prompts to update your domain’s **nameservers** to use Cloudflare
4. Once domain is active, go to the **Zero Trust Dashboard** > **Access** > **Tunnels**
5. Click **Create a Tunnel**  
   - Name: `homeassistant`  
   - Choose **Create Tunnel Token**
6. Select **Public Hostname**
   - Hostname: `ha.your-custom-domain.com`  
   - Service: `http://<your-local-device-ip-address>:8123`
7. After creating the tunnel, you’ll get a **tunnel token**.

---

## 🚀 Step 2: Install and Configure the Cloudflared Add-on

1. Go to **Settings > Add-ons > Add-on Store**
2. Search for **Cloudflared** and install the official version
3. Click **Configuration**, and enter the following:

```yaml
external_hostname: ha.your-custom-domain.com.com
additional_hosts: []
tunnel_name: homeassistant
tunnel_token: >-
  <your-token>
nginx_proxy_manager: false
```

4. Click **Save**, then **Start** the add-on

> 🧠 *Note: Do not manually manage a config file or credentials if you're using this token method—it’s handled for you.*

---

## ⚠️ Step 3: Fix “400: Bad Request” in Home Assistant

Home Assistant will reject Cloudflare traffic by default unless you tell it to trust proxy headers.

In `configuration.yaml`, add:

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - ::1
    - 172.30.33.0/24  # Docker subnet (used by Cloudflared add-on)
    - 192.168.1.0/24  # Optional: your local LAN
```

Save the file.

---

## 🔀 Step 4: Restart Home Assistant

- Go to **Settings > System > Restart**
- Wait for full reload

---

## ✅ Step 5: Test It

Visit:

```
https://ha.your-custom-domain.com
```

You should see the Home Assistant login screen—fully secure and tunneled through Cloudflare.

---

## 🧐 Optional: Secure with Cloudflare Access

- Go to **Zero Trust Dashboard > Access > Applications**
- Add an access rule for `ha.your-custom-domain.com` (e.g., email login, OTP, etc.)

---

## 👍 Troubleshooting

- **400: Bad Request** → `trusted_proxies` is missing or misconfigured
- **Tunnel won’t connect** → Check logs in the Cloudflared add-on
- **DNS not resolving** → Ensure `ha` is created in Cloudflare DNS as a CNAME to the tunnel

