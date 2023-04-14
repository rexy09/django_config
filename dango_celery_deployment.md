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

