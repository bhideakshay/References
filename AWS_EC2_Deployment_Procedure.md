I have created the `deployment_procedure.md` file with the detailed deployment steps. Here is the content of the file:

---

# Deployment Procedure

This is the file which gives step by step Deployment procedure for the AWS EC2 Deployment for the django-react-mysql project

## Reference 

[YouTube Tutorial](https://www.youtube.com/watch?v=jE7c3dDdReQ)

## Update Machine

```bash
sudo apt update
```

## Install MySQL Server

```bash
sudo apt-get install mysql-server
```

### Check SQL Status

```bash
sudo systemctl is-active mysql
```

```bash
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Isat123#';
sudo mysql_secure_installation
```

Check status:

```bash
systemctl status mysql.service
```

## Give Remote Access to MySQL Server

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
Replace `xxx.xxx.xxx.xxx` with your IP Address:

```
bind-address = xxx.xxx.xxx.xxx
```

```bash
sudo mysql -u root -p
```

```sql
CREATE USER 'django'@'localhost' IDENTIFIED BY 'Isat123#';
CREATE USER 'django'@'%' IDENTIFIED BY 'Isat123#';

-- or --

ALTER USER 'django'@'localhost' IDENTIFIED BY 'Isat123#';
ALTER USER 'django'@'%' IDENTIFIED BY 'Isat123#';

GRANT ALL ON *.* TO 'django'@'localhost';
GRANT ALL ON *.* TO 'django'@'%';

FLUSH PRIVILEGES;
EXIT;
```

```bash
sudo service mysql restart 
sudo systemctl status mysql.service
```

## Firewall Configuration

```bash
sudo ufw status verbose
```
If it returns inactive, then enable it:

```bash
sudo ufw enable
```

To disable it:

```bash
sudo ufw disable
```

Allow necessary ports:

```bash
sudo ufw allow 22
sudo ufw allow 443
sudo ufw allow 80
sudo ufw allow 3306
sudo ufw allow 8000
```

Check status:

```bash
sudo ufw status verbose
```

## Install Important Packages

```bash
sudo apt update 
sudo apt install python3-pip
sudo apt install git
sudo apt install python3-virtualenv
sudo apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config
```

Check installed packages:

```bash
pip list
```

Install Nginx:

```bash
sudo apt install nginx
sudo service nginx status
```

Check if Nginx is working by visiting your server's IP address.

## Set Up SSH Keys

```bash
pwd
cd ~/.ssh
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Change ownership of the .ssh directory:

```bash
cd ..
sudo chown -R ubuntu .ssh
```

Copy the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

## Add SSH Key to GitHub

1. Go to your GitHub repository.
2. Click on Settings.
3. Click on Deploy Keys in the sidebar.
4. Click on Add Deploy Key and paste the copied SSH public key.
5. Click on Add Key.

## Clone Project from GitHub

```bash
git clone git_repo_via_SSH
```

## Set Up Virtual Environment

```bash
virtualenv env_name
source env_name/bin/activate
```

Install dependencies:

```bash
sudo apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config
pip install -r requirements.txt
```

Install Gunicorn:

```bash
pip install gunicorn
```

Deactivate virtual environment:

```bash
deactivate
```

## Configure Gunicorn

Create and edit the socket file:

```bash
sudo vim /etc/systemd/system/testserver.gunicorn.socket
```

Add the following content:

```
[Unit]
Description=testserver.gunicorn socket

[Socket]
ListenStream=/run/testserver.gunicorn.sock

[Install]
WantedBy=sockets.target
```

Create and edit the service file:

```bash
sudo vim /etc/systemd/system/testserver.gunicorn.service
```

Add the following content:

```
[Unit]
Description=testserver.gunicorn daemon
Requires=testserver.gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/DjangoFileUploader
ExecStart=/home/ubuntu/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/testserver.gunicorn.sock \
          fileupload.wsgi:application

[Install]
WantedBy=multi-user.target
```

Start and enable Gunicorn:

```bash
sudo systemctl start testserver.gunicorn.socket
sudo systemctl start testserver.gunicorn.service
sudo systemctl enable testserver.gunicorn.socket
sudo systemctl enable testserver.gunicorn.service
```

Check status:

```bash
sudo systemctl status testserver.gunicorn.socket
sudo systemctl status testserver.gunicorn.service
```

Reload and restart Gunicorn:

```bash
sudo systemctl daemon-reload
sudo systemctl restart testserver.gunicorn
```

## Configure Nginx

Create and edit the Nginx configuration file:

```bash
sudo vim /etc/nginx/sites-available/testserver
```

Add the following content:

```
server {
    listen 80;
    listen [::]:80;

    server_name 54.86.9.16;

    location = /favicon.ico { access_log off; log_not_found off; }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://unix:/run/testserver.gunicorn.sock;
    }

    location /static/ {
        alias /home/ubuntu/DjangoFileUploader/staticfiles;
    }

    location /media/ {
        alias /home/ubuntu/DjangoFileUploader/media;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/testserver /etc/nginx/sites-enabled/testserver
```

Test Nginx configuration:

```bash
sudo nginx -t
```

Restart Nginx:

```bash
sudo service nginx restart
```

## Final Steps

Activate virtual environment again:

```bash
source env_name/bin/activate
```

Run the following Django management commands:

```bash
python manage.py clean_pyc
python manage.py clear_cache
python manage.py collectstatic
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
```

Deactivate virtual environment:

```bash
deactivate
```

Reload and restart services:

```bash
sudo systemctl daemon-reload
sudo systemctl restart testserver.gunicorn
sudo service nginx restart
```

## Permissions

```bash
sudo chmod 775 /home
sudo chmod 775 /home/ubuntu
sudo chmod -R 775 /home/ubuntu/premier
```

---

You can find this documentation in the `deployment_procedure.md` file.