# Django Project Configuration
Django project configurations

## Gitignore
### Django migrations .gitigonre
Django migrations conflict arises when there is a mismatch between the migration files in the local development environment and the production environment. This occurs when some changes are made in the local environment, and the migration files are not properly synced with the production environment.

One solution to this problem is to exclude migration files from the deployment process using a .gitignore file. The .gitignore file should include the following lines of code:

```
# Migrations files ignore example
accounts/migrations/**
dashboard/migrations/**
administration/migrations/**
*.pyc
!__init__.py
```

This will exclude all migration files, pyc files, and database files from the deployment process. It is important to note that .gitignore should only be used for files that are generated automatically and do not need to be version controlled.

To avoid migration conflicts, it is recommended to create a migration for every change made in the local environment, test the migration in a staging environment before deploying to production, and ensure that all migration files are properly synced between environments. This will ensure that the database schema remains consistent across all environments.

### Other Django .gitigonre
```
# Media files ignore 
media/**
!media/.gitkeep
!.gitkeep

staticfiles/**
db.sqlite3
.env
```
