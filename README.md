# SASSY_API
An API for the Student Attendance and Submission System. This API was developed by Keegan Crankshaw.

## Requirements
The following is required for development:  
  - Ubuntu
  - PostGres
  - Python 3
  - PyCharm (Professional version recommended - Obtainable through the Github student pack)
  
## Installation
Installation takes place in a few steps. 

1) First, install Pycharm   
    ```$ snap install pycharm-professional```  
2) Configure Postgres. Items in <> brackets are variables
    ```
    $ sudo su - postgres  
    $ psql
    $ CREATE DATABASE sassy;
    $ CREATE USER <username> WITH PASSWORD '<password>';
    $ ALTER ROLE <username> SET client_encoding TO 'utf8';
    $ ALTER ROLE <username> SET default_transaction_isolation TO 'read committed';
    $ ALTER ROLE <username> SET timezone TO 'UTC+2';
    $ GRANT ALL PRIVILEGES ON DATABASE sassy TO <username>;
    $ \q
    $ exit
    $ sudo /etc/init.d/postgresql restart
    ```
3) Pull the repository:  
    Open PyCharm and select "open from repository". Enter the following URL: ```https://github.com/kcranky/SASSY_API```
    Pycharm will take a while to index the project.

4) Set Up Pycharm
    - Open Pycharm   
    - Select File - Open and open the seat directory (~/PycharmProjects/SASSY_API)
    - Navigate to File - Settings
    - Under "Project: seat" - "Project Interpreter", ensure that the project interpreter is set to \verb| ~/PycharmProjects/SASSY_API/venv/bin/python|
    - Close and reopen PyCharm to complete the loading
  
5) Setting Configurations
    - Import ```settings.py``` to /SASSY_API/Sassy
    - In ```settings.py```, adjust the username and password for database access
    - Run ```$ python manage.py makemigrations```
    - Run ```$ python manage.py makemigrations sassy_api```
    - Run ```$ python manage.py migrate```
    
6) Create a superuser
    - Run ```python manage.py createsuperuser``` and follow the prompts. You do not need to use a legitimate email address.
    
7) Run the server   
   You can run a local development server by using ```$ python manage.py runserver```. Navigate to ```127.0.0.1:8000/api/``` in your browser to see the server running.
   
## Hosting
The API is hosted on a server.
1) SSH in
2) Create a virtual environment
3) use git to clone (into the /webapps directory)
4) Set up database as above (being sure to run migrations)
5) The settings file is available on the server. Changes are as follows:
    - Changes to debug (set to false)
    - Changes to allowed hosts (set to IP of server)
   Additions to the settings file are as follows:
    - ```USE_X_FORWARDED_HOST = True```
6) Set up gunicorn_cfg.py in the same directory as manage.py as follows:
    ``` command = '/webapps/sassy/bin/gunicorn'
    pythonpath = '/webapps/sassy/Sassy'
    bind = '127.0.0.1:8001'
    workers = 5
    user = 'nobody'
    ```
7) Edit ```/etc/nginx/sites-available/Sassy``` with the following
    ```
    server {
        listen 80;

        server_name 165.22.68.223;

        access_log off;

        location /static/ {
            alias /webapps/sassy/static/;
        }

        location / {
            proxy_pass http://127.0.0.1:8001/;
            proxy_set_header X-Forwarded-Host $server_name;
            proxy_set_header X-Real-IP $remote_addr;
            add_header P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
        }
    }
    ```
8) Run the following:
    ```
    cd /etc/nginx/sites-enabled
    sudo ln -s ../sites-available/Sassy
    sudo rm default
    sudo service nginx restart
    ```
9) Enter back into ```/webapps/sassy``` and run ```gunicorn -c gunicorn_cfg.py Sassy.wsgi```

## Resetting the DB
It shouldn't happen, but sometimes migrations become irrecoverable. Run the following:  
- drop the db and create it again
    ```
    sudo su - postgres
    psql
    drop database <sassy_db>;
    CREATE DATABASE <sassy_db>;
    GRANT ALL PRIVILEGES ON DATABASE <sassy_db> TO <username>;
    \q
    exit
    ```
- Delete all migrations 
    ```
    find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
    find . -path "*/migrations/*.pyc"  -delete
    pip install --upgrade --force-reinstall  Django
    ```
- Make and Run migrations
- Create a new superuser
    
