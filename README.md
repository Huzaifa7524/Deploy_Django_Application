# <center> Deploy Django Application using gunicorn & Nginx in Production</center>

Author: Huzaifa Tahir

Email: [huzaifatahir7524@gmail.com](mailto:huzaifatahir7524@gmail.com)

Date: March 15 2024

Social: [Github](https://github.com/Huzaifa7524) | [Linkedin](https://www.linkedin.com/in/huzaifatahir7524/)

#

### EC2 Instance Settings 

1. Create an EC2 Instance.(use Free tier,Ubuntu Linux Machine)

2. Allow all inbound and outbound Rules.`(allow SSH (port 22),HTTP (port 80),HTTPS (port 443),Django (port 8000),SMTP (port 587))`

3. Download the .pem key and connect it with an SSH client using the CMD terminal:

```shell
ssh -i "ABC.pem" ubuntu@ec2-0-0-0-0.us-west-1.compute.amazonaws.com
```

### Main  Steps

### 1. update  the linux system  and libraries

```shell
sudo apt update
sudo apt upgrade
```

### 2. Install Python and pip

```shell
sudo apt install python3-pip
```

### 3. Install Virtual env

```shell 
sudo apt install python3-virtualenv
# how to create virtualenv?
virtualenv [ Name of env ]
virtualenv myenv
# How to activate virtualenv?
source myenv/bin/activate
```

### 4. Clone Github Repo

```shell
git clone [URL]
# Then, use 'cd' to enter inside your code directory. 
```

### 5. Requirements.txt

```shell
# How to create Requirements.txt?
pip3 freeze > requirements.txt
# how to Install Requirements.txt?
pip3 install -r requirements.txt
```

### 6. Install Some Libraires

```shell
# To install Nginx server 
sudo apt install nginx
#  Installing Django and gunicorn
pip install django gunicorn

```

### 7.  Setting up our Django project

 - Add your IP address or domain to the `ALLOWED_HOSTS` variable in settings.py.

```shell
# If you have any migrations to run, perform the action:
python manage.py makemigrations
python manage.py migrate
```

### 8. Collect Static Files

```shell
python manage.py collectstatic
```

### 9. Import Thing about static files

You must make sure to add few lines in your seeting.py file.
  
  - Add this line, ``` “whitenoise.runserver_nostatic”, ``` into your `Installed_apps` of setting file.
  - Add `‘whitenoise.middleware.WhiteNoiseMiddleware’`, into MiddleWare of your setting File.
  - Also, add these lines at the bottom of the `Appname/urls. py` file.
```python
# also add this import
from django.conf import settings # new
from  django.conf.urls.static import static #new
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root = settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root = settings.STATIC_URL)
```
```python
STATIC_URL = 'static/'
STATICFILES_DIRS = [os.path.join(BASE_DIR,"static")]
STATIC_ROOT = os.path.join(BASE_DIR,"staticfiles")

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
```

### 10. Install whitenoise

```shell
pip install whitenoise
```

## <center>Gunicorn Configuration   <center>

### 1. Create a system socket file for gunicorn

```shell
sudo vim /etc/systemd/system/gunicorn.socket
```

Paste the contents below and save the file

```Shell
[Unit]
Description=gunicorn socket
[Socket]
ListenStream=/run/gunicorn.sock
[Install]
WantedBy=sockets.target
```

### 2. Create a service file for gunicorn

```shell
sudo vim /etc/systemd/system/gunicorn.service
```

Paste the contents below inside this file:

```Shell
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target
[Service]
User=ubuntu
Group=www-data
# WorkingDirectory=/home/ubuntu/[Change it with your project]
WorkingDirectory=/home/ubuntu/AI_tutor    
# ExecStart=/home/ubuntu/[Change it with  your Env]/bin/gunicorn \
ExecStart=/home/ubuntu/env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          ahsa.wsgi:application
[Install]
WantedBy=multi-user.target
```

### 3. start and enable the gunicorn socket

```shell
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

### 4. check if already a file exists Then Remove it Like [defalut].
```shell
cd /etc/nginx/sites-enabled/
# remove this 
sudo rm -rf defalut
```

## <center> Configuring Nginx as a reverse proxy </center>

### <p style='color:red;font-size:20px'> Important Note </p>

### 1. Deactivate the virtual environment 

```shell
deactivate
```

### 2. Create a configuration file for Nginx

```shell
# change name accourding to your app and also leave it same.
# sudo vim /etc/nginx/sites-available/[Your app Name]
sudo vim /etc/nginx/sites-available/AI
```

Paste the below contents inside the file created

```shell
server {
    listen 80 default_server;
    server_name _;
    location = /favicon.ico { access_log off; log_not_found off; }
    # staticfiles
    location /staticfiles/ {
        # root /home/ubuntu/[Your app  Name];
        root /home/ubuntu/AI;
    }
    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

### 3. Activate the configuration

```shell
# sudo ln -s /etc/nginx/sites-available/[your app nam] /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/AI /etc/nginx/sites-enabled/
```

### 4. Run this command to load a static file

```shell
# change user name but my username is ubuntu
sudo gpasswd -a www-data ubuntu
# sudo gpasswd -a www-data [username]
```

### 5. Restart nginx and allow the changes to take place.

```shell
sudo systemctl restart nginx
```

### 6. Activate the virtualenv 

```shell 
source myenv/bin/activate
```
and goto your working dir.

### 7. restart  gunicorn and nginx

```shell
sudo service gunicorn restart
sudo service nginx restart
```

### 8. to Reload daemon

```shell
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

### 9. Check the status of Nginx to ensure it's running without errors

```shell
sudo systemctl status nginx.service
```

### 10. Additionally in case of errors

- To check error logs
```shell
sudo tail -f /var/log/nginx/error.log
```

- To check nginx working fine
```shell
sudo systemctl status nginx
```


