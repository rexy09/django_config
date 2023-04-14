# Celery with Django in production

## Gunicorn Configuretions
First install Gunicorn inside the virtualenv:
``` bash
 pip3 install gunicorn
```

Create a file named `gunicorn_start` inside the `bin/` folder:
``` bash
nano bin/gunicorn_start_celery
```

And add the following information and save it:
`/home/{directory}/bin/gunicorn_start_celery`

``` bash
#!/bin/bash

DIR=/root/receipt/env/receipt
USER=root
GROUP=root
DJANGO_SETTINGS_MODULE=receipt.settings

cd $DIR
source ../bin/activate

export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DIR:$PYTHONPATH

exec celery -A receipt worker -B -l INFO
```

Make the gunicorn_start file is executable:
``` bash
chmod u+x bin/gunicorn_start_celery
```
Create a directory named run, for the unix socket file:
``` bash
mkdir run
```

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
sudo nano /etc/supervisor/conf.d/receipt_celery.conf
```

Example
```
[program:receipt_celery]
command=/root/receipt/env/bin/gunicorn_start_celery
user=root
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/root/receipt/env/logs/gunicorn-error.log
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
sudo supervisorctl status receipt_celery
```
```
receipt_celery                  
RUNNING   pid 23381, uptime 0:00:15
```

Now you can control your application using Supervisor. If you want to update the source code of your application with a new version, you can pull the code from GitHub and then restart the process:
```
sudo supervisorctl restart receipt_celery
```
Where receipt_celery will be the name of your process.
