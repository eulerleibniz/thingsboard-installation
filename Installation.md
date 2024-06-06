# Important Notes Before Begining Thingsboard Installation

1. The installation process should only be done by **Server Administrator**.
2. As a **Tenant Administrator** just ask your **Server Administrator** for credentials and host address. (No installation is required)
3. This tutorial is suitable for **_Local Servers Only_** (No Internet Access and No Clustering).
4. This is still pretty useful as many small-scale IoT projects fall into this category. (eg. IoT applications for a single Industrial, Commercial , and or Residential Complex)

# Steps Before Installing Thingsboard:

**_Note_**: You need to read and understand each command very well but it's generally best to Copy and Paste these commands as even a single whitespace and or hyphen can cause a lot of trouble for you.

## 1. Access the server and create a new user for administration

**_Note_**: Replace `dariush` with your username

**_Note_**: Replace `123.123.123.123` with your server's IP

Access the server for the first time (From Linux, Mac, Windows, and or any IDE that supports `ssh`):

```bash
ssh root@123.123.123.123
```

Create a new user except for `root`:

```bash
sudo adduser dariush
```

Add new user to the sudo group:

```bash
sudo usermod -aG sudo dariush
```

Switch to the new user:

```bash
su - dariush
```

From this point on you can access the server using this command:

```bash
ssh dariush@123.123.123.123
```

## 2. Update your server for bug fixes and security updates:

In the rest of this tutorial if you had any trouble downloading packages it's usually a good idea to use the main server which is a bit slow but almost always works. You can do this by editing the following file:

```bash
sudo nano /etc/apt/sources.list
```

Fully update the system (Security updates, bug fixes, and sometimes kernel updates):

```bash
sudo apt update
sudo apt upgrade
# sudo reboot
```

Upgrade the system to the next LTS release if you need. This will upgrade from 18.04 => 20.04 => 22.04 => 24.04 one at a time:

```bash
# WARNING: Only do this if you are using Linux Ubuntu 20.04 or older.
# Linux Ubuntu 24.04 is still pretty new so you better stay at Ubuntu 22.04
sudo apt update
sudo do-release-upgrade
# sudo reboot
```

## 3. Setup servers for Internet Access aiming for High Availablity (HA) and allowing Integration to External Services

In production, quite often your server needs some sort of **Internet Access** with **High Availablity (HA)** and **Integration into External Services**.

**_This tutorial won't cover these topics as it will take hours and days_**.

Here is a very short list of challenges (I completed some of these and marked them as done):

- [ ] Networking multiple instances of Thingsboard, Kafka, PostgreSQL, and Cassandra running across multiple servers (physical nodes)
- [x] Load Balancing (can be done using HAProxy or the load balancer from your CDN solution such as Cloudflare)
- [x] Configuring Ports and exposing the micro-services securely to the local network and internet as needed

- [x] Configuring the servers _Internally_ for security (disable root login, enable firewall, iptables, ...)
- [ ] Setting up unique Mail Server, SMS Server, ... and ensuring high delivery rate and low spam rate

- [x] Configuring rule engine queues

- [x] Setting up integrations for authentication services (Google, Github, ...)
- [x] Setting up integrations for messaging apps (Slack, Discord, Telegram, ...)
- [x] Setting up integrations for other APIs (Firebase, Google Maps, ...)

- [x] Ensuring the website is well exposed to the internet and search engines (Google, Bing)
- [x] Managing DNS records (Subdomains, Mail Server, Certificates, ...)
- [x] Using CDN for Improvingg Security (HTTPs, Bot Fighter, ...)
- [ ] Using CDN for Customization (Custom Pages, Regional Access, ...)
- [x] Using CDN for Performance Optimization (Caching, Route Optimization, ...)

- [ ] Monitoring server using tools provided by server provider (iranserver in my case)
- [x] Monitoring server using tools provided by CDN (Cloudflare in my case)
- [ ] Monitoring server using other custom tools to prevent server downtime and improve overall system reliability

# Installing Thingsboard

From this point on we will follow instructions in the following link:

https://thingsboard.io/docs/user-guide/install/ubuntu/

**_WARNING_**: Always use the provided link for the latest updates, I will put all the commands here just to show what they do and explain them.

- I choose Linux Ubuntu installation as it is the most widely used option.
- Installation on Raspberry Pi is usefull for very small-scale and embedded projects.
- Installation on Docker is only useful if we are looking for automating deployment and scaling using orchestration platforms such as Kubernetes.

# 1. Install Java 11 (OpenJDK)

Install Java:

```bash
sudo apt update
sudo apt install openjdk-11-jdk
```

- `sudo` stands for Super User Do. Needed for runing any commands which needs administrator privilage (even reading some files which needs super user access)
- `apt` is the software for package management and dependency installation (almost identical to `apt-get` but more user-friendly)
- `update` doesnt install anything but only checks for updates on packages cache
- always run `sudo apt update` before **_ANY STAGE_** of installation

Check Java version and configure your operating system to use OpenJDK 11 by default:

```bash
java -version
## Output should look like this:
# openjdk version "11.0.xx"
# OpenJDK Runtime Environment (...)
# OpenJDK 64-Bit Server VM (build ...)
## IF NOT:
sudo update-alternatives --config java
```

# 2. Install ThingsBoard Service

Install `wget` (stands for Web Get) if not already installed:

```bash
sudo apt update
sudo apt install -y wget
```

Use `wget`to download thingsboard binary from github

```bash
wget https://github.com/thingsboard/thingsboard/releases/download/v3.6.4/thingsboard-3.6.4.deb
```

Use `dpkg` which stands for Debian Package Manager to install .deb file

```bash
sudo apt update
sudo dpkg -i thingsboard-3.6.4.deb
```

# 3. Installing ThingsBoard database

First choose between PostgreSQL and or Hybrid (PostgreSQL+Cassandra). Here are a few tips:

- PostgreSQL (recommended for < 5K msg/sec)
- Hybrid = PostgreSQL+Cassandra (recommended for > 5K msg/sec)

- In Hybrid mode ThingsBoard will be storing timeseries data in Cassandra while continue to use PostgreSQL for main entities (devices/assets/dashboards/customers).
- Cassandra is a NoSQL database (suitable for timeseries) => Faster Storage, Faster Retrievial, Smaller Database
- Cassandra downside is that it requires lots of ram (8GB by default)

## 3.1. PostgreSQL installation

It's best to follow the documentation from https://www.postgresql.org/ website itself. Just make sure to use the correct version of the PostgreSQL as required by thingsboard.

### 3.1.1. Download and install PostgreSQL

Following this link:

https://www.postgresql.org/download/linux/ubuntu/

Scroll down to where you find following lines and copy them (Dont paste them yet):

```bash
# Import the repository signing key:
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# Create the repository configuration file:
sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'

# Update the package lists:
sudo apt update

# Install the latest version of PostgreSQL:
# If you want a specific version, use 'postgresql-16' or similar instead of 'postgresql'
sudo apt -y install postgresql-15
```

**_WARNING_**: In the last line make sure to replace `postgresql-15` with the correct version of postgresql

Now start the postgresql service:

```bash
sudo service postgresql start
```

### 3.1.2. Configuring PostgreSQL User Password and Creating the Database

**_Important_**: Commands which switch user and or switch shell should be Copy and Pasted ONE LINE AT A TIME.

Switch to the postgress user console

```bash
sudo su - postgres
```

**_Important_**: At this point we are effectively logined as `postgress` user.

Run the `psql` shell as the `postgress` user

```bash
psql
```

**_Important_**: At this point we are using `psql` shell as the `postgress` user.

Set the password for main postgresql user:

```shell
\password
```

Quit the `psql` shell:

```shell
\q
```

**_Important_**: We need to go back to the main user console. Press CTRL+D (EOF Pulse in Linux) only ONCE.

```bash
# Press Ctrl + D      (EOF Pulse in Linux)
```

Run the `psql` shell as the `main admin` user to connect to the database

```bash
psql -U postgres -d postgres -h 127.0.0.1 -W
```

- -U postgres: Specifies the username (postgres) to connect to the database.
- -d postgres: Specifies the name of the database (postgres) to connect to.
- -h 127.0.0.1: Specifies the host address (127.0.0.1 or localhost) where the PostgreSQL server is running.
- -W: Prompts for the password of the `postgres` user

Create thingsboard DB:

```shell
CREATE DATABASE thingsboard;
```

Quit the `psql` shell

```shell
\q
```

## 3.2. Cassandra installation

It's best to follow the documentation from https://cassandra.apache.org website itself. Just make sure to use the correct version of the Cassandra as required by thingsboard.

Download and install Cassandra:

```bash
# Add the Apache Cassandra repository keys:
sudo curl -o /etc/apt/keyrings/apache-cassandra.asc https://downloads.apache.org/cassandra/KEYS
# Add the Apache repository of Cassandra to /etc/apt/sources.list.d/cassandra.sources.list, for example for the latest 4.1
echo "deb [signed-by=/etc/apt/keyrings/apache-cassandra.asc] https://debian.cassandra.apache.org 41x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
# Update the repositories
sudo apt-get update
# Install Cassandra:
sudo apt-get install cassandra
```

Finally Install Cassandra Tools as Thingsboard recommends:

```bash
sudo apt-get install cassandra-tools
```

# 4. Choose ThingsBoard queue service (almost always install Kafka)

In a small project we don't need to do anything at all. But for virtually all production projects we need to setup Kafka

## 4.1. Install and Configure ZooKeeper (Required by Kafka)

Install ZooKeeper

```bash
sudo apt update
sudo apt install zookeeper
```

Create systemd unit file for Zookeeper:

```bash
sudo nano /etc/systemd/system/zookeeper.service
```

Copy and Paste the following content to the file:

```ini
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

Enable and start ZooKeeper Immediately:

```bash
sudo systemctl enable --now zookeeper
```

## 4.2. Install and Configure Kafka

**_NOTE_**: Get the correct download link for the Kafka packages from https://kafka.apache.org/ website itself

```bash
# Download the Zip file
wget https://downloads.apache.org/kafka/3.6.2/kafka_2.13-3.6.2.tgz
# Untar (unzip) the compressed file
tar xzf kafka_2.13-3.6.2.tgz
# Move the Kafka folder
sudo mv kafka_2.13-3.6.2 /usr/local/kafka
```

Create systemd unit file for Kafka:

```bash
sudo nano /etc/systemd/system/kafka.service
```

Copy and Paste the following content to the file:

```ini
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=PUT_YOUR_JAVA_PATH"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

**_Note_**: Make sure to replace “PUT_YOUR_JAVA_PATH” with your real JAVA_HOME path as per the Java installed on your system, by default like “/usr/lib/jvm/java-11-openjdk-xxx”. You can find this path using a command like this:

```bash
update-java-alternatives -l
```

Enable and start Kafka Immediately:

```bash
sudo systemctl enable --now kafka
```

# 5. ThingsBoard Configuration - PostgreSQL, Cassandra, Kafka, Memory Usage Configuration

Open the `thingsboard.conf` file using the `nano` text editor

```bash
sudo nano /etc/thingsboard/conf/thingsboard.conf
```

## 5.1. PostgreSQL Only Configuration in Thingsboard

**_If You Chose PostgreSQL Only_**: Add the following lines to the configuration file. Don’t forget to replace **_PUT_YOUR_POSTGRESQL_PASSWORD_HERE_** with your real postgres user password:

```bash
# DB Configuration
export DATABASE_TS_TYPE=sql
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=PUT_YOUR_POSTGRESQL_PASSWORD_HERE
# Specify partitioning size for timestamp key-value storage. Allowed values: DAYS, MONTHS, YEARS, INDEFINITE.
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS
```

## 5.2. Hybrid (PostgreSQL+Cassandra) Configuration in Thingsboard

**_If You Chose Hybrid (PostgreSQL+Cassandra)_**: Add the following lines to the configuration file. Don’t forget to replace **_PUT_YOUR_POSTGRESQL_PASSWORD_HERE_** with your real postgres user password:

```bash
# DB Configuration
export DATABASE_TS_TYPE=cassandra
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=PUT_YOUR_POSTGRESQL_PASSWORD_HERE
```

In case of cluster configuration, you should add the following parameters to reconfigure your ThingsBoard instance to connect to external Cassandra nodes:

```bash
export CASSANDRA_CLUSTER_NAME=Thingsboard Cluster
export CASSANDRA_KEYSPACE_NAME=thingsboard
export CASSANDRA_URL=127.0.0.1:9042
export CASSANDRA_USE_CREDENTIALS=false
export CASSANDRA_USERNAME=
export CASSANDRA_PASSWORD=
```

## 5.3. Kafka Configuration in Thingsboard

If you installed Kafka add the following parameters to reconfigure your ThingsBoard instance to connect to external Kafka nodes:

```bash
export TB_QUEUE_TYPE=kafka
export TB_KAFKA_SERVERS=localhost:9092
```

## 5.4. Update ThingsBoard memory usage and restrict it to 2G (2 Giga Bytes)

```bash
export JAVA_OPTS="$JAVA_OPTS -Xms2G -Xmx2G"
```

# 6. Run installation script for Thingsboard

First make sure all the services are installed correctly and are up and running. Run the folloing commands ONE AT A TIME.

```bash
# sudo reboot
sudo systemctl status postgresql
sudo systemctl status cassandra
sudo systemctl status zookeeper
sudo systemctl status kafka
```

If all the above services are working correctly execute the following script:

```bash
# --loadDemo option will load demo data: users, devices, assets, rules, widgets.
sudo /usr/share/thingsboard/bin/install/install.sh --loadDemo
```

Execute the following command to start ThingsBoard:

```bash
sudo service thingsboard start
```

Use the following command to check if thingsboard is running:

```bash
sudo systemctl status thingsboard
```

Once started, if you were installing everything locally, you will be able to open Web UI using the following link:

http://localhost:8080/

If you were installing on a server then you need to use the server IP and if the Domain name is set you should use the domain name.

The following default credentials are available if you have specified –loadDemo during execution of the installation script:

- System Administrator: sysadmin@thingsboard.org / sysadmin
- Tenant Administrator: tenant@thingsboard.org / tenant
- Customer User: customer@thingsboard.org / customer

# 7. Troubleshooting

ThingsBoard logs are stored in the `/var/log/thingsboard` directory.

You can list all the files and folders in a directory using the following command:

```bash
sudo ls -al /var/log/thingsboard
```

You can issue the following command to check if there are any errors on the backend side:

```bash
cat /var/log/thingsboard/thingsboard.log | grep ERROR
```

# 8. Post-installation steps
