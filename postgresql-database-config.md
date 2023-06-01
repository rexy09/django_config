# Creating the PostgreSQL Database and User

We’re going to jump right in and create a database and database user for our Django application.

By default, Postgres uses an authentication scheme called “peer authentication” for local connections. Basically, this means that if the user’s operating system username matches a valid Postgres username, that user can login with no further authentication.

During the Postgres installation, an operating system user named postgres was created to correspond to the postgres PostgreSQL administrative user. We need to use this user to perform administrative tasks. We can use sudo and pass in the username with the -u option.

Log into an interactive Postgres session by typing:
```
sudo -u postgres psql
```
You will be given a PostgreSQL prompt where we can set up our requirements.

First, create a database for your project:
```
CREATE DATABASE myproject;
```
Note: Every Postgres statement must end with a semi-colon, so make sure that your command ends with one if you are experiencing issues.

Next, create a database user for our project. Make sure to select a secure password:
```
CREATE USER myprojectuser WITH PASSWORD 'password';
```
Afterwards, we’ll modify a few of the connection parameters for the user we just created. This will speed up database operations so that the correct values do not have to be queried and set each time a connection is established.

We are setting the default encoding to UTF-8, which Django expects. We are also setting the default transaction isolation scheme to “read committed”, which blocks reads from uncommitted transactions. Lastly, we are setting the timezone. By default, our Django projects will be set to use UTC. These are all recommendations from the Django project itself:
```
ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
```
```
ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
```
```
ALTER ROLE myprojectuser SET timezone TO 'UTC';
```
Now, we can give our new user access to administer our new database:
```
GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;
```
When you are finished, exit out of the PostgreSQL prompt by typing:
```
\q
```
Postgres is now set up so that Django can connect to and manage its database information.

