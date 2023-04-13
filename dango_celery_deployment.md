# Celery with Django in production
0. Background
1. Create the dedicated user and group
2. Create the celery configuration file
3. Create the systemd file
4. Restart the server
5. References

## 0. Background
Three months ago, I deployed my react, Django project on a ubuntu server. I used celery for a task queue. Today I tried to look celery environment and I got confused about the files and my celery configuration. Now In this post, I will try to explain all the things that you should know about the celery configuration on the server.

If you never heard about celery, check out the celery official documentation, First steps with celery and First steps with Django before reading this post.

In the celery official documentation, you will find the three possible ways to daemoniz your celery with python, i.e. init-script, Init-script: celerybeat and using systemd. In this post, I will only explain about the systemd demonization.

## 1. Create the dedicated user and group
First of all, you need to create the dedicated user and group as celery,
```
sudo groupadd celery
```
```
sudo useradd -g celery celery
```

## 2. Create the celery configuration file
Now let's create the celery configuration file inside the `/etc/default/celeryd` directory,

Note: Please replace the user with your ubuntu user, `CELERY_APP_NAME`, `CELERYD_CHDIR`, and `CELERY_BIN`.

```
#   most people will only start one node:
CELERYD_NODES="worker1"
#   but you can also start multiple and configure settings
#   for each in CELERYD_OPTS
#CELERYD_NODES="worker1 worker2 worker3"
#   alternatively, you can specify the number of nodes to start:
#CELERYD_NODES=10

# Absolute or relative path to the 'celery' command:
CELERY_BIN="/home/user/.virtualenvs/venv/bin/celery"
#CELERY_BIN="/virtualenvs/def/bin/celery"

# App instance to use
# comment out this line if you don't use an app
CELERY_APP="celery_app_name"
# or fully qualified:
#CELERY_APP="proj.tasks:app"

# Where to chdir at start.
CELERYD_CHDIR="/home/user/django-project/"

# Extra command-line arguments to the worker
CELERYD_OPTS="--time-limit=300 --concurrency=8"
# Configure node-specific settings by appending node name to arguments:
#CELERYD_OPTS="--time-limit=300 -c 8 -c:worker2 4 -c:worker3 2 -Ofair:worker1"

# Set logging level to DEBUG
#CELERYD_LOG_LEVEL="DEBUG"

# %n will be replaced with the first part of the node name.
CELERYD_LOG_FILE="/var/log/celery/%n%I.log"
CELERYD_PID_FILE="/var/run/celery/%n.pid"

# Workers should run as an unprivileged user.
#   You need to create this user manually (or you can choose
#   a user/group combination that already exists (e.g., nobody).
CELERYD_USER="celery"
CELERYD_GROUP="celery"
CELERYD_LOG_LEVEL="INFO"
# If enabled PID and log directories will be created if missing,
# and owned by the userid/group configured.
CELERY_CREATE_DIRS=1
```

Now lets change the owner of the celery log and PID file,
```
chown -R celery:celery /var/log/celery/
```
```
chown -R celery:celery /var/run/celery/
```
## 3. Create the systemd file
Now lets create the systemd file inside /etc/systemd/system/celery.service directory and write following content,

Note: Please replace the user with your ubuntu user and make sure to check the path to bin celery file (this file will available probably inside your virtual environment)
```
[Unit]
Description=Celery Service
After=network.target

[Service]
Type=forking
User=celery
Group=celery

EnvironmentFile=/etc/default/celeryd
WorkingDirectory=/home/user/django-project
ExecStart=/home/user/.virtualenvs/venv/bin/celery multi start ${CELERYD_NODES} \
  -A ${CELERY_APP} --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} --loglevel=${CELERYD_LOG_LEVEL} ${CELERYD_OPTS}
ExecStop=/home/user/.virtualenvs/venv/bin/celery ${CELERY_BIN} multi stopwait ${CELERYD_NODES} \
  --pidfile=${CELERYD_PID_FILE}
ExecReload=/home/user/.virtualenvs/venv/bin/celery ${CELERY_BIN} multi restart ${CELERYD_NODES} \
  -A ${CELERY_APP} --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} --loglevel=${CELERYD_LOG_LEVEL} ${CELERYD_OPTS}

[Install]
WantedBy=multi-user.target
```
