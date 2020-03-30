# linuxServer
Linux server hosting a web application

# About
This project sets up and secures a Linux instance on a virtual machine. Then it installs and configures a web and database server to host a web application.

The application is hosted at 54.145.198.31.
The Linux distribution is Ubuntu 16.04 LTS. The virtual private server is Amazon Lighsail. The web application is an Item Catalog project. The database server is PostgreSQL. My local machine is a MacBook Pro (Mac OS X 10_14_6).

# Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail
- Login into Amazon Lightsail using an Amazon Web Services account.
- Create instance.
- Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
- Choose a plan Keep the default name provided by AWS or rename your instance.
- Click the Create button to create instance.

# Step 2: Download default private key
- SSH into the server From the Account menu on Amazon Lightsail.
- Click on SSH keys tab and download the Default Private Key.
- Move this private key file named LightsailDefaultPrivateKey-*.pem into the local folder ~/.ssh.
- Rename it lightsail_key.rsa.
- In your terminal, type: chmod 600 ~/.ssh/lightsail_key.rsa.
- To connect to the lightsail instance via the terminal: ssh -i ~/.ssh/lightsail_key.rsa ubuntu@54.145.198.31, where 54.145.198.31 is the public IP address of the instance.

# Step 3: Update and upgrade installed packages
- sudo apt-get update
- sudo apt-get upgrade

# Step 4: Change the SSH port from 22 to 2200
- Edit the /etc/ssh/sshd_config file: sudo nano /etc/ssh/sshd_config.
- Change the port number on line 5 from 22 to 2200.
- Save and exit using CTRL+X and confirm with Y.
- Restart SSH: sudo service ssh restart.

# Step 5: Configure the Uncomplicated Firewall (UFW)
Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

- sudo ufw status # The UFW should be inactive.
- sudo ufw default deny incoming # Deny any incoming traffic.
- sudo ufw default allow outgoing # Enable outgoing traffic.
- sudo ufw allow 2200/tcp # Allow incoming tcp packets on port 2200.
- sudo ufw allow www # Allow HTTP traffic in.
- sudo ufw allow 123/udp # Allow incoming udp packets on port 123.
- sudo ufw deny 22 # Deny tcp and udp packets on port 53.
- sudo ufw enable. # Turn UFW on.

The output should be like this:

Command may disrupt existing ssh connections. Proceed with operation (y|n)? y Firewall is active and enabled on system startup

Check the status of UFW to list current roles: sudo ufw status.
The output should be like this:

Status: active
To Action From

2200/tcp ALLOW Anywhere
80/tcp ALLOW Anywhere
123/udp ALLOW Anywhere
22 DENY Anywhere
2200/tcp (v6) ALLOW Anywhere (v6)
80/tcp (v6) ALLOW Anywhere (v6)
123/udp (v6) ALLOW Anywhere (v6)
22 (v6) DENY Anywhere (v6)

Exit the SSH connection: exit.

# Step 6: Change firewall configuration on lightsail to match internal firewall
- Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the internal firewall settings above.
- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

- From your local terminal, run:
ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@54.145.198.31
where 54.145.198.31 is the public IP address of the instance.

# Step 7: Give grader access
- Create a new user account named grader while logged in as ubuntu
  sudo adduser grader
- Enter a password (twice) and fill out information for this new user.

# Step 8: Give grader sudo permissions
- sudo visudo
- Search for the line that looks like this:

root ALL=(ALL:ALL) ALL

- Below this line, add the following line to give sudo privileges to grader user.

root ALL=(ALL:ALL) ALL grader ALL=(ALL:ALL) ALL

- Save and exit using CTRL+X and confirm with Y.

- Verify that grader has sudo permissions. Run su - grader, enter the password, run sudo -l and enter the password again.

# Step 9: Create an SSH key pair for grader using the ssh-keygen tool
- On the local machine: Run ssh-keygen
- Enter file in which to save the key (I gave the name grader_key) in the local directory ~/.ssh
- Enter in a passphrase twice. Two files will be generated ( ~/.ssh/grader_key and ~/.ssh/grader_key.pub)
- Run cat ~/.ssh/grader_key.pub and copy the contents of the file Log in to the grader's virtual machine
- On the grader's virtual machine: Create a new directory called ~/.ssh (mkdir .ssh)
  Run sudo nano ~/.ssh/authorized_keys and paste the content into this file, save and exit
- Give permissions: chmod 700 .ssh and chmod 644 .ssh/authorized_keys
- Check in /etc/ssh/sshd_config file if PasswordAuthentication is set to no
- Restart SSH: sudo service ssh restart
- On the local machine, run: ssh -i ~/.ssh/grader_key -p 2200 grader@54.145.198.31.

# Step 10: Prepare to deploy the project
Configure the local timezone to UTC While logged in as grader, configure the time zone: sudo dpkg-reconfigure tzdata. You should see the following:

Current default time zone: 'America/New York' Local time is now: Thu Oct 19 21:55:16 EDT 2017. Universal Time is now: Fri Oct 20 01:55:16 UTC 2017. References

# Step 11: Install and configure Apache
- Install and configure Apache to serve a Python mod_wsgi application
- While logged in as grader, install Apache: sudo apt-get install apache2.

- Enter public IP of the Amazon Lightsail instance into browser.

- Install mod_wsgi
- Run sudo apt-get install libapache2-mod-wsgi python-dev
- Enable mod_wsgi with sudo a2enmod wsgi
- Start the web server with sudo service apache2 start

# Step 12: Install and configure PostgreSQL
- While logged in as grader, install PostgreSQL: sudo apt-get install postgresql.
- PostgreSQL should not allow remote connections.
- In the /etc/postgresql/9.5/main/pg_hba.conf file, you should see:

local all   postgres              peer
local all   all                   peer
host  all   all   127.0.0.1/32    md5
host  all   all   ::1/128         md5

- Switch to the postgres user: sudo su - postgres.

- Open PostgreSQL interactive terminal with psql.

- Create the catalog user with a password and give them the ability to create databases:

postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER ROLE catalog CREATEDB;

List the existing roles: \du. The output should be like this:

                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

- Exit psql: \q.

- Switch to grader user:
  exit

- Create a new Linux user called catalog:
  sudo adduser catalog
- Enter password and fill out information.

- Give to catalog user the permission to sudo.
  Run: sudo visudo.

- Search for the following lines in file:

root ALL=(ALL:ALL) ALL
grader ALL=(ALL:ALL) ALL

- Below this line, add a new line to give sudo privileges to catalog user.

root ALL=(ALL:ALL) ALL grader ALL=(ALL:ALL) ALL catalog ALL=(ALL:ALL) ALL

- Save and exit using CTRL+X and confirm with Y.

- Verify that catalog has sudo permissions.
  Run su - catalog
  Enter the password
  Run sudo -l
  Enter the password again

- While logged in as catalog, create a database: createdb catalog.
- Run psql and then \l to confirm that the new database has been created
- Exit psql: \q.
- Switch back to the grader user: exit

# Step 13: Deploy Item Catalog project
- In the /var/www directory, change ownership of catalog directory to grader:
  sudo chown -R grader:grader catalog/
- Change to the /var/www/catalog/catalog directory.
- Rename application.py file to __init__.py using:
  mv application.py __init__.py

- In __init__.py, replace line:
app.run(host="0.0.0.0", port=8000, debug=True)
with:
app.run()

- In database.py, replace line:
  engine = create_engine("sqlite:///catalog.db")
  with:
  engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')

# Step 14: Install the virtual environment and its dependencies
- While logged in as grader, install pip: sudo apt-get install python3-pip.
- Install the virtual environment: sudo apt-get install python-virtualenv
- Change to the /var/www/catalog/catalog/ directory.
- Create the virtual environment: sudo virtualenv -p python3 venv3.
- Change the ownership to grader with: sudo chown -R grader:grader venv3/.
- Activate the new environment: . venv3/bin/activate.
- Install the following dependencies:
  pip install httplib2
  pip install requests
  pip install --upgrade oauth2client
  pip install sqlalchemy
  pip install flask
  sudo apt-get install libpq-dev
  pip install psycopg2

# Step 15: Run the app and see the site
Run python __init__.py and the site comes up here: http://54.145.198.31/
