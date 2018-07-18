# linux-server #
In this project, I set up a linux server on an AWS Lightsail Ubuntu machine. The server is running Apache, and the web application is based on Flask and PostgreSQL. In order to access the website, please go to http://18.191.16.213.xip.io/. The URL is 18.191.16.213.

----
## Summary of Installed Software ##
These are the main packages that needed to be installed on the server. I found that installing Python libraries via `sudo apt install python3-[package]` was more reliable than Pip, because Pip installed the Python packages only to my user directory and then Python had a hard time detecting them when running a file located in the `/var/www/` directory.
```
apache2
libapache2-mod-wsgi-py3
python3-pip
python3-flask
python3-sqlalchemy
python3-psycopg2
```
---------------
## Configurations
Several configurations needed to be made to this server before it was functional. These are described below.

### Acquire Lightsail Machine
Open an AWS account [here](aws.amazon.com), and create an OS only machine with Ubuntu. Start by going to the Networking tab and making sure the following four ports are allowed: 22/tcp, 2200/tcp, 123/udp, and 80/tcp.


### Install updates to your system
Run these commands and then reboot. Repeat this process until you login to your machine and it says that you have 0 updates remaining.

```
sudo apt update
sudo apt full-upgrade
```


### Configure and enable UFW
Allow only three specific ports, enable the `ufw`, and finally restart the ssh service.

```
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw allow 2200/tcp
sudo ufw enable
sudo service ssh restart
```


### Change SSH port from 22 to 2200
Edit `/etc/ssh/ssdh_config` and change `Port 22` to `Port 2200`. Restart SSH via `sudo service ssh restart`.


### Configure the time zone
Run `sudo dpkg-reconfigure tzdata`


### Create the `grader` user and give it sudo privileges
Connect to your Lightsail machine using the built-in browser SSH window and run the following commands. We will set the password for `grader` to be `grader`.

```
sudo adduser grader
sudo usermod -aG sudo grader
```


### Add an SSH key for the `grader` user
Run `ssh-keygen` on a local machine, outputting to the default location. Back on the server, we will have to temporarily allow password authentication for SSH because our `grader` account has a password and we will need that to set up the SSH key.

Therefore, edit `/etc/ssh/sshd_config` and look for a block that says,
```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
``` 

Change it to `PasswordAuthentication yes`. Back on the client machine, run the following in order to copy the SSH key to the server, and enter the password when prompted.
```
ssh-copy-id grader@18.191.16.213 -p 2200
```

Now go back to the `sshd_config` file and disable clear text passwords. Finally, restart the SSH service as above.


### Configure Apache
Note: in this section, all packages are assumed to be already installed, as noted in the Package Summary section above.

1. Clone the `item-catalog` git repo into `/var/www/`
2. Create a `Catalog.conf` file in `/etc/apache2/sites-available`:

    ```apacheconf
    <VirtualHost *>

        ServerName 18.191.16.213
        WSGIScriptAlias / /var/www/catalog/Catalog.wsgi
        WSGIDaemonProcess catalog-server

        <Directory /var/www/catalog>

                WSGIProcessGroup catalog-server
                WSGIApplicationGroup %{GLOBAL}
                Order deny,allow
                Allow from all

        </Directory>

    </VirtualHost>
    ```
3. Create `Catalog.wsgi` in `/var/www/catalog/`:

    ```python
    import sys
    sys.path.insert(0, '/var/www/catalog')
    from n_final import app as application
    ```
 
4. We're going to be using xip.io to help create a Google OAuth-compatible URL, so go to the Google Developer Console of the app and add http://18.191.16.213.xip.io to the authorized redirect URIs and Authorized JavaScript origins.


### Setup PostgreSQL Database

1. Run `su - postgres` and `psql` to enter into the Postgres console.
2. Create the `catalog` database.
3. Create the `catalog` user and grant it all privileges on the `catalog` database.
4. Alter `catalog` to have the `password` as its password.
5. Run `\q` and `exit` to get back to bash.
6. Edit `/etc/postgresql/9.5/main/pg_hba.conf` and find the line that says:

    ```
    # Database administrative login by Unix domain socket
    local   all             postgres                                peer
    ```

    Change the `peer` to `md5` so we can connect to the database with the created username and password.

7. Edit `n_database.py` and `n_final.py` in `var/www/catalog` and find the line that says:

    ```python
    engine = create_engine([...SQLite...])
    ```

    Replace that line with 
    
    ```python
    engine = create_engine('postgresql://catalog:password@localhost/catalog')
    ```

### Final Steps
Now we need to disable to default Apache website and enable our newly created configuration, and finally restart the Apache server. Run the following commands to accomplish this.

```
sudo a2dissite 000-default.conf
sudo a2ensite Catalog.conf
sudo service apache2 restart
```

Now everything should be configured and working properly, and we can access the site at http://18.191.16.213.xip.io/


--------
## Bibliography
1. https://wiki.apache.org/httpd/ClientDeniedByServerConfiguration
2. https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9
3. https://stackoverflow.com/questions/38298652/permissionerror-errno-13-permission-denied-flask-run
4. https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
5. https://github.com/hicham-alaoui/ha-linux-server-config