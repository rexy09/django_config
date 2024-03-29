# Installing the Packages from the Ubuntu Repositories

To begin the process, we’ll download and install all of the items we need from the Ubuntu repositories. We will use the Python package manager pip to install additional components a bit later.

We need to update the local apt package index and then download and install the packages. The packages we install depend on which version of Python your project will use.

If you are using Django with Python 3, type:

```
sudo apt update
```
### Installing python3 Libraries

```
sudo apt install python3-pip python3-dev libpq-dev
```
### Installing Nginx

```
sudo apt install nginx curl
```
### Installing PostgreSQL
```
sudo apt install postgresql postgresql-contrib
```
## Creating a Python Virtual Environment

Now that we have our database, we can begin getting the rest of our project requirements ready. We will be installing our Python requirements within a virtual environment for easier management.

To do this, we first need access to the virtualenv command. We can install this with pip.

If you are using Python 3, upgrade pip and install the package by typing:

```
sudo -H pip3 install --upgrade pip
```
```
sudo -H pip3 install virtualenv
```

Within the project directory, create a Python virtual environment by typing:
```
virtualenv env
```
