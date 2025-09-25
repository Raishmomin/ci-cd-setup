# Set-up Cicd for node js, nextjs and react js using npm

## 1. Upload or clone your app
##### Example: clone from git

```
sudo git clone https://github.com/yourusername/yourapp.git
cd yourapp
```

## 2. Install dependencies

##### Install node & npm if not already installed
```
sudo apt update
sudo apt install -y nodejs npm
```

##### Install app dependencies
```
npm install --production
```

For Python (example with Flask/Django):
```
sudo apt update
sudo apt install -y python3 python3-venv python3-pip

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## 3. Create a systemd service

Create and edit a service file:
```
sudo nano /etc/systemd/system/[yourapp].service
```

Paste something like (Node.js example):
```
[Unit]
Description=[YourApp Service]
After=network.target

[Service]
ExecStart=/usr/bin/node /var/www/[yourapp]/app.js
Restart=always
User=www-data
Group=www-data
Environment=NODE_ENV=production
WorkingDirectory=/var/www/[yourapp]

[Install]
WantedBy=multi-user.target
```

For Python/Flask (gunicorn example):
```
[Unit]
Description=Gunicorn instance to serve YourApp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/yourapp
Environment="PATH=/var/www/yourapp/venv/bin"
ExecStart=/var/www/yourapp/venv/bin/gunicorn --workers 3 --bind 0.0.0.0:8000 wsgi:app
Restart=always

[Install]
WantedBy=multi-user.target
```

## 4. Reload and start service
Reload systemd to pick up new service
```
sudo systemctl daemon-reload
```

Enable service on boot
```
sudo systemctl enable [yourapp]
```

Start service now
```
sudo systemctl start [yourapp]
```

Check logs
```
sudo journalctl -u [yourapp] -f
```

## 5. (Optional) Reverse proxy with Nginx

If you want your app on port 80/443:

```
sudo nano /etc/nginx/sites-available/yourapp
```

Example config:
```
server {
    listen 80;
    server_name [yourdomain.com];

    location / {
    proxy_pass http://127.0.0.1:[3000];   # adjust if your app runs on a different port
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Fix 504 timeout
    proxy_connect_timeout 60s;
    proxy_send_timeout 300s;
    proxy_read_timeout 300s;
    send_timeout 300s;
    }
}
```

Enable site:

```
sudo ln -s /etc/nginx/sites-available/[yourapp] /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 6. Enabled Firewall

```
# Allow HTTP (80)
sudo ufw allow 80/tcp

# Allow HTTPS (443)
sudo ufw allow 443/tcp

# Reload firewall
sudo ufw reload

# Check rules
sudo ufw status


```
## 7. Check EC2 Security Group

```
Go to AWS EC2 → Security Groups → Inbound rules.

Make sure HTTP (port 80) and HTTPS (port 443) are open to 0.0.0.0/0 (anywhere).

Example rule:

Type	Protocol	Port Range	Source
HTTP	TCP	          80	0.0.0.0/0
HTTPS	TCP	         443	0.0.0.0/0
```

## 8. Setup SSL with Certbot

```
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d [yourdomain.com]

```
