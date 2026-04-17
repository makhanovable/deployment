Step 1 - Configure Django for Production
Edit `practice/settings.py`:
```python
# Add your server IP or domain
ALLOWED_HOSTS = ['your_server_ip']
```
---
Step 2 - Update the Server
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv nginx git
```
---
Step 3 - Clone the Repository
```bash
cd /root
git clone https://github.com/YOUR_REPO
cd YOUR_REPO
```
---
Step 4 - Set Up a Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
pip install django gunicorn
```
---
Step 5 - Create a Gunicorn Systemd Service
```bash
sudo nano /etc/systemd/system/gunicorn.service
```
```ini
[Unit]
Description=Gunicorn daemon for Django
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=/root/YOUR_REPO
ExecStart=/root/YOUR_REPO/venv/bin/gunicorn \
          --workers 3 \
          --bind unix:/root/YOUR_REPO/gunicorn.sock \
          practice.wsgi:application

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn

# Verify it's running
sudo systemctl status gunicorn
```
---
Step 6 - Configure Nginx
```bash
sudo nano /etc/nginx/sites-available/YOUR_REPO
```
```nginx
server {
    listen 80;
    server_name your_server_ip;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        root /home/root/YOUR_REPO;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/root/YOUR_REPO/gunicorn.sock;
    }
}
```
Enable the config and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/YOUR_REPO /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```
---
Step 7 - Open Firewall Ports
```bash
sudo ufw allow 'Nginx Full'
sudo ufw enable
```
---
Verification
Open `http://your_server_ip` in a browser — your Django app should be live.
