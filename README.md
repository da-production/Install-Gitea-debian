<p align="center">
  <a href="https://gitea.io/">
    <img alt="Gitea" src="https://raw.githubusercontent.com/go-gitea/gitea/main/public/img/gitea.svg" width="220"/>
  </a>
</p>
<h1 align="center">Install Gitea Git service on Debian 11| Debian 10</h1>

<p align="center">
In this tutorial, we’ll do installation of Gitea on Debian 11| Debian 10 Linux and configure Nginx proxy to forward requests to Gitea internal service on a port. With Nginx, you can optionally terminal SSL certificates when doing a secure setup on Gitea on Debian 10 Server.
</p>

# Step 1: Update System and Install git

You need to have git installed in your Debian machine. Let’s update our OS and ensure git is installed.

```
sudo apt -y update
sudo apt -y install git vim bash-completion
```

View the version of Git installed.

```
$ git --version
git version 2.20.1
```

# Step 2: Add git user account for Gitea

Gitea should have a dedicated local user account for management operations. Add the user and group to your Debian system by running the following commands.

```
sudo adduser \
   --system \
   --shell /bin/bash \
   --gecos 'Git Version Control' \
   --group \
   --disabled-password \
   --home /home/git \
   git
```

The user creation will assign a unique ID for the user and create its home directory.

```
Adding system user `git' (UID 108) ...
Adding new group `git' (GID 114) ...
Adding new user `git' (UID 108) with group `git' ...
Creating home directory `/home/git' ...
```

# Step 3: Install MariaDB database server

Data will be stored on MariaDB database server.

```
sudo apt -y install mariadb-server
```

Secure your database installation by setting root password and removing test database and users.

```
$ sudo mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

Create a database for Gitea.

```
$ sudo mysql -u root -p
CREATE DATABASE gitea;
GRANT ALL PRIVILEGES ON gitea.* TO 'gitea'@'localhost' IDENTIFIED BY "StrongP@ssword";
FLUSH PRIVILEGES;
QUIT;
```

# Step 4: Install Gitea on Debian 11| Debian 10

The gitea binary packages are available on the Downloads page. Check the latest release before downloading it.

```
curl -s  https://api.github.com/repos/go-gitea/gitea/releases/latest |grep browser_download_url  |  cut -d '"' -f 4  | grep '\linux-amd64$' | wget -i -
```

Move the downloaded binary file to the /use/local/bin

```
chmod +x gitea-*-linux-amd64
sudo mv gitea-*-linux-amd64 /usr/local/bin/gitea
```

Confirm successful installation by checking the version of Gitea installed.

```
$ gitea --version
Gitea version 1.14.4 built with GNU Make 4.1, go1.16.5 : bindata, sqlite, sqlite_unlock_notify
```

# Step 5: Configure Systemd

Create directories required for Gitea setup.

```
sudo mkdir -p /etc/gitea /var/lib/gitea/{custom,data,indexers,public,log}
sudo chown git:git /var/lib/gitea/{data,indexers,log}
sudo chmod 750 /var/lib/gitea/{data,indexers,log}
sudo chown root:git /etc/gitea
sudo chmod 770 /etc/gitea
```

The web installer will need write permission configuration file under /etc/gitea

### Create a systemd service file for Gitea.

```
sudo vim /etc/systemd/system/gitea.service
```

Configure the file and set User, Group and WorkDir.

```
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target
After=mysql.service

[Service]
LimitMEMLOCK=infinity
LimitNOFILE=65535
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web -c /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
```

Reload systemd and restart Gitea service.

```
sudo systemctl daemon-reload
sudo systemctl enable --now gitea
```

Check service status.

```
$ systemctl status gitea
● gitea.service - Gitea (Git with a cup of tea)
   Loaded: loaded (/etc/systemd/system/gitea.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2019-10-20 07:37:06 UTC; 27s ago
 Main PID: 2637 (gitea)
    Tasks: 9 (limit: 4719)
   Memory: 93.8M
   CGroup: /system.slice/gitea.service
           └─2637 /usr/local/bin/gitea web -c /etc/gitea/app.ini

Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 routers/init.go:74:GlobalInit() [T] Custom path: /var/lib/gitea/custom
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 routers/init.go:75:GlobalInit() [T] Log path: /var/lib/gitea/log
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 ...dules/setting/log.go:226:newLogService() [I] Gitea v1.9.4 built with GNU Make 4.1, go1.12
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 ...dules/setting/log.go:269:newLogService() [I] Gitea Log Mode: Console(Console:info)
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 ...les/setting/cache.go:42:newCacheService() [I] Cache Service Enabled
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 ...s/setting/session.go:45:newSessionService() [I] Session Service Enabled
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 routers/init.go:106:GlobalInit() [I] SQLite3 Supported
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 routers/init.go:37:checkRunMode() [I] Run Mode: Development
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 cmd/web.go:151:runWeb() [I] Listen: http://0.0.0.0:3000
Oct 20 07:37:06 deb10 gitea[2637]: 2019/10/20 07:37:06 ...ce/gracehttp/http.go:142:Serve() [I] Serving [::]:3000 with pid 2637
```

# Step 6: Configure (Nginx / Apache) proxy

[Nginx](https://docs.gitea.io/en-us/reverse-proxies/#nginx)
[Apache](https://docs.gitea.io/en-us/reverse-proxies/#apache-httpd)

If ufw is enabled, allow http and https ports.
```
for i in http https; do
 sudo ufw allow $i
done
```

and enable proxy module

# Step 7: Finish Gitea Installation from Web interface
After configuring (Nginx / Apache) proxy, access Gitea web interface on http://servername/install
