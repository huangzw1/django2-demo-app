Django 2 demo app (ubuntu 16.04)
========================

**Note!** mod_wsgi must be installed without activating Python virtual environment in current session.
It is not possible for the one mod_wsgi instance to run applications for both Python 2 and 3 at the same time.

## Python and CentOS problem 
CentOS ships with Python as a critical part of the base system. 
Because it is a critical part it is not getting updated, other than to plug security vulnerabilities.
mod-wsgi is available only for system default Python version.

## Python on Ubuntu 16.04

```
sudo apt-get install python-pip
pip install --upgrade pip
pip install virtualenv
virtualenv py3.5 --python=python3.5
source py3.5/bin/activate
python -V
pip -V
pip install django
deactivate
sudo apt-get install libapache2-mod-wsgi-py3
```

## Test deployment checklist

```
python manage.py check --deploy
```

## Check virual environment path
```
python -c 'import sys; print(sys.prefix)'
```

## Collect static files
```
./manage.py collectstatic
```

## Run dev server
```
./manage.py runserver 0.0.0.0:8000
```

And go [127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)

## Locate "site-packages" dir
```
(my_env) [vagrant@localhost mysite]$ python
Python 3.4.5 (default, Dec 11 2017, 14:22:24) 
>>> import django.core.wsgi
>>> import os
>>> os.path.dirname(django.core.wsgi.__file__)
```

## Deploy with Apache and wsgi_module

Check that "mod_wsgi" apache module is installed and enabled. 
```
apache2ctl -M | grep wsgi
```

Create file /etc/apache2/sites-available/django2-mysite.conf

```
sudo a2ensite django2-mysite
```

Put basic configuration in django.mysite.conf:

```
<VirtualHost *:80>
    ServerName django2-mysite.dev
    ErrorLog /var/log/apache2/django2-mysite.dev-error_log
    CustomLog /var/log/apache2/django2-mysite.dev-access_log common
    LogLevel notice
    
    WSGIScriptAlias / /vagrant/django/mysite/mysite/wsgi.py
    WSGIDaemonProcess mysite processes=2 threads=15 display-name=%{GROUP} python-home=/home/vagrant/py3.5 python-path=/vagrant/django/mysite
    WSGIProcessGroup mysite
    
    Alias /static /vagrant/django/mysite/static
    
    <Directory /home/user/myproject/static>
        Require all granted
    </Directory>
    
    <Directory /vagrant/django/mysite/mysite>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
</VirtualHost>
```

Where "mysite" is the project name (not website's name).

See [Django 2 docs](https://docs.djangoproject.com/en/2.0/howto/deployment/wsgi/modwsgi/)

Other docs about [WSGIDaemonProcess](http://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIDaemonProcess.html)

## Python implementation check
```
python -c 'import platform; print(platform.python_implementation())'
```