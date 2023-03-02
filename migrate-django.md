# How to migrate Django from SQLite to PostgreSQL

Dump existing data:
```
python3 manage.py dumpdata > datadump.json
```

Change settings.py to Postgres backend.

Make sure you can connect on PostgreSQL. Then:
```
python3 manage.py migrate --run-syncdb
```

Run this on Django shell to exclude contentype data
```
python3 manage.py shell
```
```
>>> from django.contrib.contenttypes.models import ContentType
>>> ContentType.objects.all().delete()
>>> quit()
````

Finally:
```
python3 manage.py loaddata datadump.json
```

> Source: https://stackoverflow.com/questions/3034910/whats-the-best-way-to-migrate-a-django-db-from-sqlite-to-mysql