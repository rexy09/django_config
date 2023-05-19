# Django Deployment Steps

## Gunicorn Configuretions (Option 1)
First install Gunicorn inside the virtualenv:
``` bash
 pip3 install gunicorn
```

Create a file named `gunicorn_start` inside the `bin/` folder:
``` bash
nano bin/gunicorn_start
```

And add the following information and save it:
`/home/{directory}/bin/gunicorn_start`

``` bash
#!/bin/bash

NAME="vanelo"
DIR=/home/vanelo/api/vanelo
USER=vanelo
GROUP=vanelo
WORKERS=3
BIND=unix:/home/vanelo/run/gunicorn.sock
DJANGO_SETTINGS_MODULE=vanelo.settings
DJANGO_WSGI_MODULE=vanelo.wsgi
LOG_LEVEL=error

cd $DIR
source ../bin/activate

export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DIR:$PYTHONPATH

exec ../bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $WORKERS \
  --user=$USER \
  --group=$GROUP \
  --bind=$BIND \
  --log-level=$LOG_LEVEL \
  --log-file=-
```

Make the gunicorn_start file is executable:
``` bash
chmod u+x bin/gunicorn_start
```
Create a directory named run, for the unix socket file:
``` bash
mkdir run
```

## Creating systemd Socket and Service Files for Gunicorn (Option 2)
We have tested that Gunicorn can interact with our Django application, but we should implement a more robust way of starting and stopping the application server. To accomplish this, we’ll make systemd service and socket files.

The Gunicorn socket will be created at boot and will listen for connections. When a connection occurs, systemd will automatically start the Gunicorn process to handle the connection.

### 1. Start by creating and opening a systemd socket file for Gunicorn with sudo privileges:
``` bash
sudo nano /etc/systemd/system/gunicorn.socket
```

Inside, we will create a [Unit] section to describe the socket, a [Socket] section to define the socket location, and an [Install] section to make sure the socket is created at the right time:
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```
Save and close the file when you are finished.

### 2. Next, create and open a systemd service file for Gunicorn with sudo privileges in your text editor. The service filename should match the socket filename with the exception of the extension:
``` bash
sudo nano /etc/systemd/system/gunicorn.service
```

Start with the [Unit] section, which is used to specify metadata and dependencies. We’ll put a description of our service here and tell the init system to only start this after the networking target has been reached. Because our service relies on the socket from the socket file, we need to include a Requires directive to indicate that relationship:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target
```
Next, we’ll open up the [Service] section. We’ll specify the user and group that we want to process to run under. We will give our regular user account ownership of the process since it owns all of the relevant files. We’ll give group ownership to the www-data group so that Nginx can communicate easily with Gunicorn.
We’ll then map out the working directory and specify the command to use to start the service. In this case, we’ll have to specify the full path to the Gunicorn executable, which is installed within our virtual environment. We will bind the process to the Unix socket we created within the /run directory so that the process can communicate with Nginx. We log all data to standard output so that the journald process can collect the Gunicorn logs. We can also specify any optional Gunicorn tweaks here. For example, we specified 3 worker processes in this case:
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myprojectdir
ExecStart=/home/sammy/myprojectdir/myprojectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          myproject.wsgi:application
```
Finally, we’ll add an [Install] section. This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:
### Final
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myprojectdir
ExecStart=/home/sammy/myprojectdir/myprojectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          myproject.wsgi:application

[Install]
WantedBy=multi-user.target
```

With that, our systemd service file is complete. Save and close it now.

We can now start and enable the Gunicorn socket. This will create the socket file at `/run/gunicorn.sock` now and at boot. When a connection is made to that socket, systemd will automatically start the gunicorn.service to handle it:

```
sudo systemctl start gunicorn.socket
```
```
sudo systemctl enable gunicorn.socket
```
We can confirm that the operation was successful by checking for the socket file.

### Checking for the Gunicorn Socket File
Check the status of the process to find out whether it was able to start:
``` bash
sudo systemctl status gunicorn.socket
```

You should receive an output like this:
``` bash
You should receive an output like this:
  gunicorn.socket - gunicorn socket
       Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor preset: enabled)
       Active: active (listening) since Thu 2022-11-10 12:08:12 UTC; 2min 2s ago
     Triggers: ● gunicorn.service
       Listen: /run/gunicorn.sock (Stream)
       CGroup: /system.slice/gunicorn.socket

  Nov 10 12:08:12 vanelo-staging systemd[1]: Listening on gunicorn socket.
  ```
  
Next, check for the existence of the gunicorn.sock file within the /run directory:
```
file /run/gunicorn.sock
```
#### Output
```
/run/gunicorn.sock: socket
```

If the systemctl status command indicated that an error occurred or if you do not find the gunicorn.sock file in the directory, it’s an indication that the Gunicorn socket was not able to be created correctly. Check the Gunicorn socket’s logs by typing:
```
sudo journalctl -u gunicorn.socket
```

Take another look at your `/etc/systemd/system/gunicorn.socket` file to fix any problems before continuing.

### Testing Socket Activation
Currently, if you’ve only started the gunicorn.socket unit, the gunicorn.service will not be active yet since the socket has not yet received any connections. You can check this by typing:
``` bash
sudo systemctl status gunicorn
```
#### Output
```
● gunicorn.service - gunicorn daemon
   Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```

To test the socket activation mechanism, we can send a connection to the socket through curl by typing:
``` bsah
curl --unix-socket /run/gunicorn.sock localhost
```
You should receive the HTML output from your application in the terminal. This indicates that Gunicorn was started and was able to serve your Django application. You can verify that the Gunicorn service is running by typing:
```
sudo systemctl status gunicorn
```
```
 gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2020-06-26 18:52:21 UTC; 2s ago
TriggeredBy: ● gunicorn.socket
   Main PID: 22914 (gunicorn)
      Tasks: 4 (limit: 1137)
     Memory: 89.1M
     CGroup: /system.slice/gunicorn.service
             ├─22914 /home/sammy/myprojectdir/myprojectenv/bin/python /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunico>
             ├─22927 /home/sammy/myprojectdir/myprojectenv/bin/python /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunico>
             ├─22928 /home/sammy/myprojectdir/myprojectenv/bin/python /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunico>
             └─22929 /home/sammy/myprojectdir/myprojectenv/bin/python /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunico>

Jun 26 18:52:21 django-tutorial systemd[1]: Started gunicorn daemon.
Jun 26 18:52:21 django-tutorial gunicorn[22914]: [2020-06-26 18:52:21 +0000] [22914] [INFO] Starting gunicorn 20.0.4
Jun 26 18:52:21 django-tutorial gunicorn[22914]: [2020-06-26 18:52:21 +0000] [22914] [INFO] Listening at: unix:/run/gunicorn.sock (22914)
Jun 26 18:52:21 django-tutorial gunicorn[22914]: [2020-06-26 18:52:21 +0000] [22914] [INFO] Using worker: sync
Jun 26 18:52:21 django-tutorial gunicorn[22927]: [2020-06-26 18:52:21 +0000] [22927] [INFO] Booting worker with pid: 22927
Jun 26 18:52:21 django-tutorial gunicorn[22928]: [2020-06-26 18:52:21 +0000] [22928] [INFO] Booting worker with pid: 22928
Jun 26 18:52:21 django-tutorial gunicorn[22929]: [2020-06-26 18:52:21 +0000] [22929] [INFO] Booting worker with pid: 22929
```

If the output from curl or the output of systemctl status indicates that a problem occurred, check the logs for additional details:
```
sudo journalctl -u gunicorn
```
Check your `/etc/systemd/system/gunicorn.service` file for problems. If you make changes to the /etc/systemd/system/gunicorn.service file, reload the daemon to reread the service definition and restart the Gunicorn process by typing:
```
sudo systemctl daemon-reload
```
```
sudo systemctl restart gunicorn
```
Make sure you troubleshoot the above issues before continuing.

## Configure Supervisor
Supervisor will start the application server and manage it in case of server crash or restart:
```
sudo apt-get -y install supervisor
```

Enable and start the Supervisor:
```
sudo systemctl enable supervisor
```
```
sudo systemctl start supervisor
```

Now what we want to do is configure **Supervisor** to take care of running the gunicorn server.
First let’s create a folder named **logs** inside the virtualenv:
```
mkdir logs
```

Create a file to be used to log the application errors:
```
touch logs/gunicorn-error.log
```

Create a new Supervisor configuration file:
```
sudo nano /etc/supervisor/conf.d/vanelo.conf
```

Example
```
[program:vanelo]
command=/home/vanelo/api/bin/gunicorn_start
user=vanelo
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/home/vanelo/api/logs/gunicorn-error.log
```

Reread Supervisor configuration files and make the new program available:
```
sudo supervisorctl reread
```
```
sudo supervisorctl update
```

Check the status:
```
sudo supervisorctl status vanelo
```
```
vanelo                  
RUNNING   pid 23381, uptime 0:00:15
```

Now you can control your application using Supervisor. If you want to update the source code of your application with a new version, you can pull the code from GitHub and then restart the process:
```
sudo supervisorctl restart vanelo
```
Where vanelo will be the name of your application.

## Configure Nginx to Proxy Pass to Gunicorn
Now that Gunicorn is set up, we need to configure Nginx to pass traffic to the process.
Start by creating and opening a new server block in Nginx’s sites-available directory:
```
sudo nano /etc/nginx/sites-available/myproject
```
Inside, open up a new server block. We will start by specifying that this block should listen on the normal port 80 and that it should respond to our server’s domain name or IP address:
```
server {
    listen 80;
    server_name server_domain_or_IP;
}
```
Next, we will tell Nginx to ignore any problems with finding a favicon. We will also tell it where to find the static assets that we collected in our `~/myprojectdir/static` directory. All of these files have a standard URI prefix of “/static”, so we can create a location block to match those requests:

```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/sammy/myprojectdir;
    }
}
```
Finally, we’ll create a location / {} block to match all other requests. Inside of this location, we’ll include the standard proxy_params file included with the Nginx installation and then we will pass the traffic directly to the Gunicorn socket:
```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/sammy/myprojectdir;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```
Save and close the file when you are finished. Now, we can enable the file by linking it to the sites-enabled directory:

### Create logs files
```
mkdir logs
```
```
touch logs/nginx-access.log
```
```
touch logs/nginx-error.log
```

#### Example 1
```
upstream app_server {
    server unix:/run/gunicorn.sock fail_timeout=0;
}

server {

    # add here the ip address of your server
    # or a domain pointing to that ip (like example.com or www.example.com)
    server_name vanelo-api.ellipsis-dev.com;

    keepalive_timeout 5;
    client_max_body_size 4G;

    access_log /home/vanelo/api/logs/nginx-access.log;
    error_log /home/vanelo/api/logs/nginx-error.log;

    location /static/ {
        alias /home/vanelo/api/vanelo/staticfiles/;
    }

    # checks for static file, if not found proxy to app
    location / {
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;
      proxy_pass http://app_server;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/vanelo-api.ellipsis-dev.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/vanelo-api.ellipsis-dev.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


server {
    if ($host = vanelo-api.ellipsis-dev.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name vanelo-api.ellipsis-dev.com;
    return 301 https://$host$request_uri; # managed by Certbot


}
```
#### Example 2
```
upstream app_server {
    server unix:/run/gunicorn.sock fail_timeout=0;
}

server {
    
    # add here the ip address of your server
    # or a domain pointing to that ip (like example.com or www.example.com)
    server_name 139.162.249.20 api.risitibox.com;

    keepalive_timeout 5;
    client_max_body_size 4G;

    access_log /root/receipt/env/logs/nginx-access.log;
    error_log /root/receipt/env/logs/nginx-error.log;
    

    location /static/ {
        alias /root/receipt/env/receipt/staticfiles/;
    }

    location /media/ {
        alias /root/receipt/env/receipt/media/;
    }


    # checks for static file, if not found proxy to app

    location / {
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;
      proxy_pass http://app_server;
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/api.risitibox.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.risitibox.com/privkey.pem; 

 # managed by Certbot
}


server {
    if ($host = api.risitibox.com) {
        return 301 https://$host$request_uri;
    } 


    listen 80;
    server_name api.risitibox.com;
    return 301 https://$host$request_uri; 

}

```

```
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```
Test your Nginx configuration for syntax errors by typing:
```
sudo nginx -t
```
If no errors are reported, go ahead and restart Nginx by typing:
```
sudo systemctl restart nginx
```
Finally, we need to open up our firewall to normal traffic on port 80. Since we no longer need access to the development server, we can remove the rule to open port 8000 as well:
```
sudo ufw delete allow 8000
```
```
sudo ufw allow 'Nginx Full'
```

You should now be able to go to your server’s domain or IP address to view your application.

Note: After configuring Nginx, the next step should be securing traffic to the server using SSL/TLS. This is important because without it, all information, including passwords are sent over the network in plain text.
If you have a domain name, the easiest way to get an SSL certificate to secure your traffic is using Let’s Encrypt. Follow this guide to set up Let’s Encrypt with Nginx on Ubuntu 20.04. Follow the procedure using the Nginx server block we created in this guide.

## Conclusion
In this guide, we’ve set up a Django project in its own virtual environment. We’ve configured Gunicorn to translate client requests so that Django can handle them. 

Afterwards, we set up Nginx to act as a reverse proxy to handle client connections and serve the correct project depending on the client request.

Django makes creating projects and applications simple by providing many of the common pieces, allowing you to focus on the unique elements. By leveraging the general tool chain described in this article, you can easily serve the applications you create from a single server.

Restart NGINX:
```
sudo service nginx restart
```

## Nginx connet to .sock failed (13:Permission denied) - 502 bad gateway

What I simply did was changing the name of the user on the first line in `/etc/nginx/nginx.conf` file.

In my case the default user was `www-data` and I changed it to my `root` machine username.
