# django_deployment
<!-- GETTING STARTED -->
## Django Deployment on Linux Server

USE CASE: Rest API or Full Django Application deployment on production server through gunicorn.

### Prerequisites

* Python3.6 or higher
  ```sh
  sudo apt-get update
  sudo apt install software-properties-common
  sudo add-apt-repository ppa:deadsnakes/ppa
  sudo apt-get update
  sudo apt-get install python3.6
  ```

* Pip3
  ```sh
  sudo apt-get install python3-pip
  ```

* Virtualenv 
  ```sh
  sudo pip3 install virtualenv
  ```

* Nginx 
  ```sh
  sudo apt-get install nginx
  ```

* Screen
  ```sh
  sudo apt-get install screen
  ```

### Building and Serving the API / Full Django Application


1. Clone the repo
   ```sh
   git clone https://github.com/your_username_/Project-Name.git
   ```
2. cd into project directory where <b>manage.py</b> and <b>gunicorn_starter.sh</b> exist

3. Activate virtualenv
   ```sh
   source env/bin/activate
   ```
   If no virtualenv exist, create a new one
   ```sh
   virtualenv env
   ```

3. Install application dependencies
   ```sh
   pip3 install -r requirements 
   ```

4. For GeoDjango application that use GDAL geospatial library 
   ```sh
   sudo apt-get install python-gdal
   ```
   
4. Config gunicorn_starter.sh
   ```sh
   gunicorn core.wsgi -b 0.0.0.0:5000 -w 5 --max-requests 100 --bind unix:/tmp/myproject.sock
   ```

5. Create screen session
   ```sh
   # change SCREEN_NAME to some unique label according to the project
   screen -S SCREEN_NAME
   ```

6. Attach to a screen session
   ```sh
   # change SCREEN_NAME to the screen session that you want to use
   screen -r SCREEN_NAME
   ```

7. Execute django application through gunicorn
   ```sh
   bash gunicorn_starter.sh
   ```

8. This will create socket object in <b>/tmp</b> which will be used to pass proxy to nginx

9. Detach from screen session
   ```sh
   ctrl + a + d
   ```

### Hosting the App
1. cd <b>/etc/nginx/sites-available/</b>

2. Write a new server block inside sites-available
   ```sh
   server {
           listen 3000 default_server;
           listen [::]:3000 default_server;
   
           # change DOMAIN_NAME into DNS that is routed to the ip 
           server_name DOMAIN_NAME;

           location = /favicon.ico { access_log off; log_not_found off; }

           location / {
                include proxy_params;
                proxy_pass http://unix:/tmp/myproject.sock;
           }
   }
   ```

3. Activate the new sites available
   ```sh
   # change NEW_SERVER_BLOCK with serverblock file name
   ln -s /etc/nginx/sites-available/NEW_SERVER_BLOCK /etc/nginx/sites-enabled/
   ```


4. Check syntax error on server block
   ```sh
   nginx -t  
   ```

5. If there is no error, proceed to restart nginx service
   ```sh
   sudo systemctl restart nginx  
   or
   sudo service nginx restart
   ```

### Updating Source Codes and Redeploy
1. cd into project directory

2. Pull latest changes from repository
   ```sh
   git pull      
   ```

3. Attach to screen session that has the application running
   ```sh
   screen -r <SCREEN>    
   ```

4. Kill and restart the django application
   ```sh
   ctrl + c
   bash gunicorn_starter.sh
   ```
