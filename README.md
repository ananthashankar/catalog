    1. IP Address and SSH Port:
     - IP Address: 52.33.117.46
     - SSH Port: 2200

    2. Complete URL:
     - http://52.33.117.46/

    3. Summary of software installed and configuration changes:

     - Important Commands to update packages
        - cat /etc/apt/sources.list -  packages in root
        - sudo apt-get update - update packages info
        - sudo apt-get upgrade - update packages
        - man apt-get - important
        - sudo apt-get autoremove - remove unwanted packages

     - Adding New User Grader
      - sudo adduser grader (password - grader)
      
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
         sudo pip install flask-seasurf
         sudo apt-get install libpq-dev python-dev
         sudo pip install python-psycopg2
         sudo pip install Flask
         python __init__.py
         deactivate
         sudo nano /etc/apache2/sites-available/catalog.conf
         Paste:
    	  <VirtualHost *:80>
    	      ServerName 52.33.117.46
    	      ServerAdmin admin@52.33.117.46
    	      ServerAlias ec2-52-33-117-46.us-west-2.compute.amazonaws.com
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

    	  from catalog import app as application
          application.secret_key = 'Add your secret key'
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
MIIEowIBAAKCAQEAzK6fGSF0yQ7D9H8aBS6IOCOsSvLUsCloOtHaAtN90J1Z+cWU
7UaltlIVNR0vdVJ/S9yPxAa8k1QmuLxXGoqFfjrPeEfqhEr57j6HEvdBvmWZTLIB
ZmDFW+E5ku+yQmLf8LHwRFUn9WdVAcY9H/dHA2UL6ufet8db8kP6i/WD9QmTdQSq
eq/rExXlKE/ezBVCXfl+Q53M8PSsi7Lbg1ig8SYT/yfvbTBKXCO/S1UkHW/J6Zwr
xOw3CuHEMzsu68uxvDE/LJWn2JAu7mioAjlfVbEentRDqxoHwSPEElkvNMRFnbmJ
TisDxMKj0I5O07DnrpRmAqKXC35S80h+dfty+wIDAQABAoIBAD2BF2Oozvv/iNh2
PO5jriEYbxRSZaDNwHk0R8tjm8HNFpVcTsUB3perkJ3WOEWL1Z6JF1YzJAUtWzlV
tuLNzxFAQMmG6qx4DyQM+++yBrpcszT9pDgMSiGyyuchSbJzHZGpFmaiJBC0zTFs
TT/GwTr+6RbcN+uHZ1SkIqxdyRofDbaP1LUZ+ZKq22qqVckzBXjeZO5zQ4D4GRv3
hes3x6FOXcW305yzDKK4JLAUdYwnJTwxZ2slIN43zC/MtJD/iXeKzVpN+ctArw5P
5dxfetSlqKh/9xcDJoL0aNzDf8hpj/atX6qRbi3w+f/JC5OoOdA3QPZoW0KSDSf+
oi6P2FECgYEA8sRlhNDEX19fffSvkaiicKtdbr82r6JMNghpbONsJ1gx4WqMgcoE
lTdqVkE9GLUcU7mWecONB12IiVqoLwh7c2XcLIX1jAP8Qm/5NId0XZZyfFtC6IP/
8Vi7ReFzdBp5xZuHMKpkior+XYQgUXWxbdIgyWzLTarZJsSVAk/ga0kCgYEA19bI
AE3POVQbuHEn/2hB8fNz/FGjFPwJw3TPZNbjxGv2aA88ru1gY/A1f6feEkoRLAVS
zPjm6g9h6IZud1GPrNu66aLMniyAIm3V5fM0tZFti5PiRUMFlJfRi2wbCO6e/f89
y1yJWlwirZVApIGUjHUur45ZXUfghOCxjwjeiCMCgYAzQoDlEGfGc47oO5guu1rB
S43I6psTbsOEzTXlhge6LwcpP6Q3a36YO0E6wT+zTdqTWyaIw1+t5HQF/Jxygen/
LczVodt9GwJSzO3jx44sjK3T0DlKe0S5ozC3yqjkJQr9TJ+5COF912dqO5HPYXh7
ZdkCbvRmi+KaKvwDpvYN8QKBgFh68c4+F38W3a5EpPLs2GvJM3jyNnp5v77iecqK
1SBGaeLKrEPBh8wwQp4sQLsapeN34zOnrXGyEJ7zzQEY7F8eTIdOd7c34uc6Q39a
rfbowRGA9DcUfIsnmX0gOgz1VTQmmDxvmNb0AjtKfg9yF2Vk/Fh3cGbu+jk+q0tJ
hYAlAoGBANeZMquXbnshgOOf07n1JAAM20I8chZHAplmcMmxpFH+sr2vHFXtCB3G
1y39g9MFkf1zvuwyn2+nlnULUu+MVdd8ArJ+YzQUURUwbv9bOgoa9Kl5DuNsd62W
qkJeT2IRAS2/6gFxXQRrZWxdTi/K/4dfr1tsjXO4LWMsBvvrnH9m
-----END RSA PRIVATE KEY-----











