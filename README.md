```markdown
# Google Cloud Setup: Docker, Nginx & Certbot for n8n

## Step 1: Update the system
```bash
sudo apt update && sudo apt upgrade -y
```

## Step 2: Create and activate a 2GB swap file
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## Step 3: Make swap permanent
```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Step 4: Verify swap activation
```bash
free -h
```

## Step 5: Install Docker
```bash
sudo apt install docker.io -y
```

## Step 6: Start and enable Docker service
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

## Step 7: Run N8N Docker Container  with Volume Persistence
```bash
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="n8nevano.ddns.net" \
-e WEBHOOK_TUNNEL_URL="https://n8nevano.ddns.net/" \
-e WEBHOOK_URL="https://n8nevano.ddns.net/" \
-v n8n_data:/home/node/.n8n \
n8nio/n8n
```

## Step 8: Install Nginx
```bash
sudo apt install nginx -y
```

## Step 9: Create an Nginx configuration file for N8N
```bash
sudo nano /etc/nginx/sites-available/n8n
```
Add the following content:
```nginx
server {
    listen 80;
    server_name n8nevano.ddns.net;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;

        # Headers for WebSocket support
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Upgrade $http_upgrade;

        # Additional headers for forwarding client info
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save and exit using:
```bash
CTRL+O, ENTER, CTRL+X
```

## Step 10: Enable the Nginx configuration
```bash
sudo rm /etc/nginx/sites-enabled/n8n
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
```

## Step 11: Test Nginx configuration
```bash
sudo nginx -t
```

## Step 12: Restart Nginx service
```bash
sudo systemctl restart nginx
```

## Step 13: Install Certbot for SSL Certificates
```bash
sudo apt install certbot python3-certbot-nginx -y
```

## Step 14: Obtain SSL Certificate
```bash
sudo certbot --nginx -d n8nevano.ddns.net
```

## Step 15: Restart Nginx service
```bash
sudo systemctl restart nginx
```
```

