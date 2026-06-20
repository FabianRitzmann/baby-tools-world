## Deployment on a Server with Nginx (Production)
 
This section describes how to deploy the application on a Linux V-Server using Gunicorn and Nginx.
 
### Prerequisites
 
- A Linux V-Server (e.g. Ubuntu 24)
- Nginx installed and running
- Python 3.12+ installed
- SSH access to the server
### 1. Clone the Repository
 
```bash
cd /home/<your-username>
git clone https://github.com/FabianRitzmann/baby-tools-world.git
cd baby-tools-world
git checkout add-product-tags
```
 
### 2. Create Virtual Environment & Install Dependencies
 
```bash
python3 -m venv my-venv
source my-venv/bin/activate
pip install -r requirements.txt
```
 
### 3. Configure Environment Variables
 
```bash
cp example.env .env
nano .env
```
 
Set the following values:
 
```
ALLOWED_HOSTS=127.0.0.1,localhost,<your-server-ip>
DEBUG=false
AUTHOR=YourName
```
 
> [!IMPORTANT]
> Make sure there are no spaces between the comma-separated values in `ALLOWED_HOSTS`.
 
### 4. Prepare the Database & Static Files
 
```bash
cd src
python manage.py migrate
python manage.py collectstatic
```
 
### 5. Set Up Gunicorn as a Systemd Service
 
Create a service file so Gunicorn starts automatically:
 
```bash
sudo nano /etc/systemd/system/babytools.service
```
 
Paste the following content:
 
```ini
[Unit]
Description=Baby Tools World
After=network.target
 
[Service]
User=<your-username>
WorkingDirectory=/home/<your-username>/baby-tools-world/src
ExecStart=/home/<your-username>/baby-tools-world/my-venv/bin/gunicorn --bind 127.0.0.1:8000 --workers 3 btw_app.wsgi:application
Restart=always
 
[Install]
WantedBy=multi-user.target
```
 
Enable and start the service:
 
```bash
sudo systemctl daemon-reload
sudo systemctl enable babytools
sudo systemctl start babytools
sudo systemctl status babytools
```
 
### 6. Configure Nginx
 
Create an Nginx configuration file:
 
```bash
sudo nano /etc/nginx/sites-available/babytools
```
 
Paste the following content:
 
```nginx
server {
    listen 80;
    server_name <your-server-ip>;
 
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
 
    location /static/ {
        alias /home/<your-username>/baby-tools-world/src/staticfiles/;
    }
 
    location /media/ {
        alias /home/<your-username>/baby-tools-world/src/media/;
    }
}
```
 
Activate the configuration and restart Nginx:
 
```bash
sudo ln -s /etc/nginx/sites-available/babytools /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```
 
### 7. Verify the Deployment
 
Open your browser and visit:
 
```
http://<your-server-ip>
```
 
The Baby Tools World application should now be running in production.
 
---