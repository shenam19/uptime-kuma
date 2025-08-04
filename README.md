# Uptime Kuma Docker Setup Guide

## HTTP Setup (Quick Start)

### Step 1: Run Uptime Kuma Container

```bash
docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data -v /var/run/docker.sock:/var/run/docker.sock --name uptime-kuma louislam/uptime-kuma:1
```

### Step 2: Access Uptime Kuma

Uptime Kuma is now running on `http://<Your IP address>:3001`

---

## HTTPS Setup (Complete Configuration)

### Step 1: Start Container with Port Mapping

```bash
docker run -d \
  --restart=always \
  -p 127.0.0.1:3001:3001 \
  -v uptime-kuma:/app/data \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name uptime-kuma \
  louislam/uptime-kuma:1
```

### Step 2: Install Nginx

```bash
sudo apt update
sudo apt install nginx
```

### Step 3: Create SSL Directory and Certificate

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/nginx/ssl/uptime-kuma.key \
    -out /etc/nginx/ssl/uptime-kuma.crt \
    -subj "/C=IN/ST=State/L=City/O=Organization/OU=IT Department/CN=<your_ip>"
```

**Note:**  For "Common Name" enter your server IP (should match your domain/hostname)

### Step 4: Create Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/uptime-kuma
```

Paste this configuration (replace `<your_ip>` with your actual server IP):

```nginx
server {
    listen 443 ssl;
    server_name <your_ip>;

    ssl_certificate /etc/nginx/ssl/uptime-kuma.crt;
    ssl_certificate_key /etc/nginx/ssl/uptime-kuma.key;

    location / {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name <your_ip>;

    location / {
        proxy_pass http://127.0.0.1:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Save and exit (`Ctrl+X`, then `Y`, then `Enter`)

### Step 5: Enable the Configuration

```bash
sudo ln -s /etc/nginx/sites-available/uptime-kuma /etc/nginx/sites-enabled/
```

### Step 6: Remove Default Nginx Site (Optional)

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Step 7: Test and Restart Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Step 8: Verify Everything Works

```bash
# Check if Uptime Kuma is accessible
curl http://127.0.0.1:3001

# Check if Nginx is running
sudo systemctl status nginx
```

### Step 9: Access Your Site

Open your browser and go to: `https://<your_server_ip>`

---

## Important Notes

- The HTTPS setup uses a self-signed certificate, so your browser will show a security warning. You can safely proceed for internal/development use.
- For production environments, consider using Let's Encrypt for a trusted SSL certificate.
- Make sure your firewall allows traffic on ports 80 (HTTP) and 443 (HTTPS).
- The Docker socket mounting (`/var/run/docker.sock`) allows Uptime Kuma to monitor Docker containers.

## Troubleshooting

- If the container fails to start, check Docker logs: `docker logs uptime-kuma`
- If Nginx fails to start, check the configuration: `sudo nginx -t`
- Ensure no other services are using ports 80, 443, or 3001
- Check firewall settings if you can't access the web interface