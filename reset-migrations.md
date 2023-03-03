# Django Reset Migrations
The Django migration system was developed and optmized to work with large number of migrations. Generally you shouldn’t mind to keep a big amount of models migrations in your code base. Even though sometimes it causes some undesired effects, like consuming much time while running the tests. But in scenarios like this you can easily disable the migrations (although there is no built-in option for that at the moment).

Anyway, if you want to perform a clean-up, I will present a few options in this tutorial.

## Scenario 1:
The project is still in the development environment and you want to perform a full clean up. You don’t mind throwing the whole database away.
1. Remove the all migrations files within your project
Go through each of your projects apps migration folder and remove everything inside, except the `__init__.py` file.
Or if you are using a unix-like OS you can run the following script (inside your project dir):
```
find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
```
```
find . -path "*/migrations/*.pyc"  -delete
```

2. Drop the current database, or delete the db.sqlite3 if it is your case.
3. Create the initial migrations and generate the database schema:
```
python manage.py makemigrations
```
```
python manage.py migrate
```
And you are good to go.

