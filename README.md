    1. IP Address and SSH Port:
     - IP Address: 127.0.0.1
     - SSH Port: 2200

    2. Complete URL:
     - http://localhost:8080/

    3. Summary of software installed and configuration changes:

     - Important Commands to update packages
        - cat /etc/apt/sources.list -  packages in root
        - sudo apt-get update - update packages info
        - sudo apt-get upgrade - update packages
        - man apt-get - important
        - sudo apt-get autoremove - remove unwanted packages

     - Adding New User Grader
      - sudo adduser grader
      
     - For Key Login
      - ssh grader@127.0.0.1 -p 2200  -i ~/.ssh/Ubuntu_Linux_Web_Server

     - Give sudo permission to user
        sudo /etc/sudoers
        sudo cat /etc/sudoers
        sudo ls /etc/sudoers.d
        sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader
        sudo nano /etc/sudoers.d/grader

     - Change SSH port from 22 to 2200
        sudo nano /etc/ssh/sshd_config - change port value for ssh
        Port to 2200
        PermitRootLogin no
        PasswordAuthentication Yes

        Restart SSH
        sudo /etc/init.d/ssh restart

     - Configuring SSH login
        mkdir .ssh - enable ssh-keygen login
        touch .ssh/authorized_keys
        nano .ssh/authorized_keys
        chmod 700 .ssh
        chmod 644 .ssh/authorized_key
        sudo nano /etc/ssh/sshd_config - PasswordAuthentication No

     - To resolve local warning on sudo access
        change value in /etc/hosts to add localhost name “localhost”
        
     - UFW Settings

        sudo ufw status
        sudo ufw default deny incoming
        sudo ufw default allow outgoing
        sudo ufw status
        sudo ufw allow ssh
        sudo ufw allow 2200/tcp
        sudo ufw allow www
        sudo ufw enable
        sudo ufw status

     - Settings to monitor unsuccessful login attempts
        
        sudo apt-get install fail2ban
        sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
        sudo vim /etc/fail2ban/jail.local
          bantime = 1000 - anytime bantam needed could be set
          action = %(action_mwl)s
          for ssh port = 2200
        sudo apt-get install sendmail iptables-persistent
        sudo iptables -S
        sudo service fail2ban stop
        sudo service fail2ban start

     - Configuring local time zone to UTC
        sudo dpkg-reconfigure tzdata
        choose None of the above and then UTC

     - Installing Apache2
        sudo apt-get install apache2
        Install python apps of apache
        sudo apt-get install python-setuptools libapache2-mod-wsgi
        sudo service apache2 restart

        To remove the local warning on restart
         echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf
         sudo a2enconf fqdn

     - Git installation and configuration
        sudo apt-get install git
        git config --global user.name "YOUR NAME"
        git config --global user.email "YOUR EMAIL ADDRESS"

     - Setting catalog application in virtual environment
        Install and Enable mod_wsgi
         sudo apt-get install libapache2-mod-wsgi python-dev
         sudo a2enmod wsgi
         
         cd /var/www
         sudo mkdir catalog
         cd catalog
         sudo mkdir catalog
         cd catalog
         sudo mkdir static templates
         sudo nano __init__.py
         Paste:
    	
    	  from flask import Flask  
      	  app = Flask(__name__)  
      	  @app.route("/")  
      	  def hello():  
        	    return “Test Good“  
      	  if __name__ == "__main__":  
        	  app.run() 

        Install flask
         sudo apt-get install python-pip
         sudo pip install virtualenv
         sudo virtualenv venv
         source venv/bin/activate
         pip install Flask
         python __init__.py
         deactivate
         sudo nano /etc/apache2/sites-available/catalog.conf
         Paste:
    	  <VirtualHost *:80>
    	      ServerName 127.0.0.1
    	      ServerAdmin admin@127.0.0.1
    	      ServerAlias localhost
    	      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    	      <Directory /var/www/catalog/catalog/>
    	          Order allow,deny
    	          Allow from all
    	      </Directory>
    	      Alias /static /var/www/catalog/catalog/static
    	      <Directory /var/www/catalog/catalog/static/>
    	          Order allow,deny
    	          Allow from all
    	      </Directory>
    	      ErrorLog ${APACHE_LOG_DIR}/error.log
     	     LogLevel warn
    	      CustomLog ${APACHE_LOG_DIR}/access.log combined
    	  </VirtualHost>

        sudo a2ensite catalog
        cd /var/www/catalog
        sudo vim catalog.wsgi
        Paste:
    	  #!/usr/bin/python
    	  import sys
    	  import logging
    	  logging.basicConfig(stream=sys.stderr)
    	  sys.path.insert(0,"/var/www/catalog/")

    	  from catalog import app as ‘super_secret_key’
        sudo service apache2 restart

    - Cloning the catalog app from the github
        git clone https://github.com/ananthashankar/catalog.git
        mv catalog /var/www/catalog/catalog/

        Setting repository to be inaccessible by web
        cd /var/www/catalog/
        sudo vim .htaccess
        RedirectMatch 404 /\.git
        
        Install supporting packages
         source venv/bin/activate
         pip install httplib2
         pip install requests
         sudo pip install --upgrade oauth2client
         sudo pip install sqlalchemy
         sudo apt-get install python-psycopg2

    - PostgreSQL installation and settings
        sudo apt-get install postgresql postgresql-contrib
        sudo vim /etc/postgresql/9.3/main/pg_hba.conf
        sudo vim database_setup.py
         change engine value to ‘create_engine('postgresql://catalog:catalog@localhost/catalog')’
        sudo vim finalproject.py
         change engine value to ‘create_engine('postgresql://catalog:catalog@localhost/catalog')’
         replace client_secrets.json with /var/www/catalog/catalog/client_secrets.json
         replace fb_client_secrets.json with /var/www/catalog/catalog/fb_client_secrets.json

        mv finalproject.py __init__.py
        User and DB creation and access settings
         sudo adduser catalog - set a password(catalog)
         sudo su - postgre
         psql
         CREATE USER catalog WITH PASSWORD ‘catalog’;
         ALTER USER catalog CREATEDB;
         CREATE DATABASE catalog WITH OWNER catalog;
         \c catalog
         REVOKE ALL ON SCHEMA public FROM public;
         GRANT ALL ON SCHEMA public TO catalog;
         \q
         exit

        Create Database
         python database_setup.py

    - Enable oauth settings for +gsign and Facebook
        
        go to developer console of the respective services and update the public and redirect url
        if Facebook setting is in developer mode then turn that off to be available for public
        
    - Monitor Glances Installation
     
        sudo apt-get install python-pip build-essential python-dev
        sudo pip install Glances
        sudo pip install PySensors

    References:

    	http://stackoverflow.com
    	https://www.digitalocean.com/community/tutorials
    	http://askubuntu.com
    	http://glances.readthedocs.org/en/latest/glances-doc.html#introduction


            

    4. RSA file content

    -----BEGIN RSA PRIVATE KEY-----
    Proc-Type: 4,ENCRYPTED
    DEK-Info: AES-128-CBC,2D1BE19DB5A5EA6B2ABF919A03290785

    SxaL4TwGYgBO6gnVh8SjN+mas1b1OzB7YkqpkzLwkKlNe1o8DcN313ZcgsdQ0ca6
    JM1zhjRUv3NDyHluPzs98vcYIdhbhl9izu6uIeuGeEPuzPDQt/hlatfKp+tF/xOE
    voheyCQ3JhkIpTA0c7578BK8pUXH6AbKTlgLUGLBnPaydKoqExO87ImQxrFegCNx
    AtBUzMQLrE3iXJ1sltw+02H8+dubXXycedZusXhsBJ/0pFEExTbv1Cumy4meICaZ
    Jel+o27Bylclwccow/v8scOuKO7uyBnB8HX71BTrYy/mDEpmFxk7IW5cM6v2IRl2
    fmsGTVI8u1hm4CMuzc3AYujfY7EiM57S9RZvezXawWcwIj1dQj6D0ZveW/yng+6N
    Bl5n7CAkHI7qdsLg6RMoayQQy22+ysPvYdI4MZJx/OmRcZNYJVGuuQYQJ4b+orW9
    bzUeS+wk/c2/UOcNKDGJCBX63/ePkd+GDfk4auVUZk2LfqWJWGRaWId+nBfNbetW
    1YhzLy0Oc/znPlwZi6JROWYUiB3cbNZSj3k+PqGxnT25MrfLGmM7Sc96Q1n7YBgb
    H/3P3sIRUF0uqHs4+UFLTDfATLw6jFiN3LoQMKwrjuUWUJbQnpZtqc0EEEWWnW0q
    OZZJcrt7+9tlLq6uRPcbyKACHTa/gffK/bI5XNgPu8cGQ/NhVE5AyuOTemecUzK1
    a7JmjNE8YXxP1OM8z+oYYFAk8sn7NewmxI/hQ/l5y+Yt81XYEO/8UQM+u5CwJ4rJ
    xSUacyIbc+dgOcpEtEkwdHpJyV2R2E+OKpMXJ3cNPYC8azCGSbEmhbBFPAGwOnJ8
    rXlaVIHAknSVo+qYiCZXzkf66yOKAP8fvkKtEpSVI9g1BDAO92WazZcIdWEzwB+P
    ORqUwqj+Y5G8iJKrPZpMuEX2pysLE+SFL7OnNFdNKzhPJ7UwQQ3qgfRs3GGh+NiP
    vYvpijSJpUKrkVPg2RdMDVHjAy7lDfstTAQCPyKoLvlzYCAHzHPJxO5Hd5136iHU
    7ROsX5HGOie+iTEsAXfbsciJXCw44VZ+EJYM0ZQr5iZF4qCzzcNZ9pWe+BdB+sNJ
    JrARBu39kftnCKisS8gUxLCxBU//QcyjMUf57JGQ+/VAHp8NIEYT9t3o4h1/AyQx
    qVfFCZ2Ce/qLR+Ik40YtCXdAlhnAXlmtAgZO1HzMzPTM8zhKbwR0txu7fRFVsr7f
    gvPMDkIhkcLXJTmPExz982L036YUMGgjROpSBAniuueLT0/8xi2I3afTrFUFrRZN
    YFwfqHqbEzZAsxoAorgEP2phfC7n6Bkm7UKabN0WMRStGvAzRMul9X4PwF1Aae32
    d9GKUoTQ9PrptAm6JnhNsoEHt0Zyid7PlsLvYn2bGC+9aqNAg0T6RiynkRjInhYq
    p5DYZFiMx0+l9uP+cv/CRE1Ry+Z+8KBok/3y4ai4042azOBB4F20sU+PX5cr0M6K
    qWLtUifBXZFY5BOFGvwPyVDK6JBgT0KruUEguoo4kcLJD1nN6ON+rpFlH7Fe5Nyl
    ti/Bqx1aMrq+eIEg6a32a2upWL8qcf9TkUpOmAqip7Pu33Zo6zrqtwghMAe7Mh0/
    -----END RSA PRIVATE KEY-----












