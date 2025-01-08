## Gitea Ubuntu 22.04 Installation

**Prepare PostgreSQL**

edit postgres configuration file /etc/postgresql/16/main/postgresql.conf
```
listen_addresses = 'localhost, 203.0.113.3'
password_encryption = scram-sha-256

```
create a postgres user and database (gitea &  giteadb)
```
sudo -u postgres psql
postgres=# CREATE ROLE gitea WITH LOGIN PASSWORD 'gitea';
postgres=# CREATE DATABASE giteadb WITH OWNER gitea TEMPLATE template0 ENCODING UTF8 LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8';
postgres=#\q
```
edit pg_hba di /etc/postgresql/16/main/pg_hba.conf, add this line 
```
local    giteadb    gitea    scram-sha-256
```
save and restart postgres 
```
sudo systemctl restart postgresql
```


**Download Binary**

```
wget -O gitea https://dl.gitea.com/gitea/1.22.6/gitea-1.22.6-linux-amd64
chmod +x gitea
```
create a new user git 
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

create directories structure

```
sudo mkdir -p /var/lib/gitea/{custom,data,log}
sudo chown -R git:git /var/lib/gitea/
sudo chmod -R 750 /var/lib/gitea/
sudo mkdir /etc/gitea
sudo chown root:git /etc/gitea
sudo chmod 770 /etc/gitea
```
move gitea binary 

```
sudo mv gitea /usr/local/bin/gitea
```
create file service (Ubuntu 22.04)

```
sudo /etc/systemd/system/gitea.service
```

The file content
```
[Unit]
Description=Gitea (Git with a cup of tea)
After=network.target
Wants=postgresql.service
After=postgresql.service
[Service]
LimitNOFILE=524288:524288
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea
[Install]
WantedBy=multi-user.target
```
save the file and run

```
sudo systemctl start gitea
sudo systemctl status gitea
```
open browser and go to URL http://localhost:3000, follow step by step web-based final installation.

if everything goes well then
```
sudo systemctl enable gitea
```


