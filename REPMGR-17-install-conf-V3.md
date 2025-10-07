# 🚀 REPMGR 17 Installation & Configuration Guide

Set up a **PostgreSQL 17 High Availability Cluster** with REPMGR!  
_Follow these steps for a robust, production-ready setup._

---

## 🖥️ Environment

- **Primary:** `postgresql-a` / `10.171.176.14`
- **Standby:** `postgresql-b` / `10.171.176.15`
- **Witness:** `postgresql-q` / `10.171.43.198`

**Requirements:**
- Red Hat Enterprise Linux 9.5
- PostgreSQL 17 installed on all nodes
- Network connectivity between all nodes
- Sudo/root access

---

## ⚙️ Preparation

### 1️⃣ Allow `postgres` User to Manage PostgreSQL Service

On **all nodes**, add this to the sudoers file so `postgres` can manage PostgreSQL without a password:

```bash
sudo visudo
```
Add:
```
postgres ALL=NOPASSWD: /usr/bin/systemctl start postgresql-17.service, /usr/bin/systemctl stop postgresql-17.service, /usr/bin/systemctl restart postgresql-17.service, /usr/bin/systemctl reload postgresql-17.service, /usr/bin/systemctl status postgresql-17.service
```

### 2️⃣ Create the `repmgr` User

On **all nodes**:
```bash
su - postgres
createuser -s repmgr
```

---

## 📦 Install REPMGR

On **all nodes** as root:
```bash
sudo dnf install -y repmgr_17.x86_64
```

---

## 🏆 Primary Node Setup (`postgresql-a`)

### 1️⃣ Configure `/etc/repmgr.conf`
```ini
node_id=1
node_name=postgresql-a
conninfo='host=postgresql-a user=repmgr dbname=userdb-repmgr connect_timeout=2'
data_directory='/pg/data01/'
pg_basebackup_options='--waldir /pg/wal'
pg_bindir='/usr/pgsql-17/bin'
failover=automatic
reconnect_attempts=2
reconnect_interval=1
location='dc1'
promote_command='/usr/pgsql-17/bin/repmgr -f /etc/repmgr.conf standby promote --log-to-file'
follow_command='/usr/pgsql-17/bin/repmgr -f /etc/repmgr.conf standby follow --log-to-file --upstream-node-id=2'
log_file='/pg/log/repmgr.log'
service_start_command   = 'sudo systemctl start postgresql-17.service'
service_stop_command    = 'sudo systemctl stop postgresql-17.service'
service_restart_command = 'sudo systemctl restart postgresql-17.service'
service_reload_command  = 'sudo systemctl reload postgresql-17.service'
```

### 2️⃣ Edit `postgresql.conf`
```conf
listen_addresses = '*'
shared_preload_libraries = 'repmgr'
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on
wal_log_hints = on
archive_mode = on
archive_command = 'cp %p /pg/wal/%f'
```

### 3️⃣ Edit `pg_hba.conf`
```conf
# repmgr
local   replication   repmgr                                trust
host    replication   repmgr            127.0.0.1/32        trust
host    replication   repmgr            all                 trust
host    all           repmgr            postgresql-a        trust
host    all           repmgr            postgresql-b        trust
host    all           repmgr            postgresql-q        trust
# repmgr database
local   userdb-repmgr repmgr                                trust
host    userdb-repmgr repmgr            127.0.0.1/32        trust
host    userdb-repmgr repmgr            postgresql-a        trust
host    userdb-repmgr repmgr            postgresql-b        trust
host    userdb-repmgr repmgr            postgresql-q        trust
```

### 4️⃣ Restart PostgreSQL
```bash
sudo systemctl restart postgresql-17.service
```

### 5️⃣ Create the REPMGR Database
```bash
su - postgres
createdb userdb-repmgr -O repmgr
```

### 6️⃣ Register the REPMGR Extension
```bash
psql
CREATE EXTENSION repmgr;
\c userdb-repmgr
CREATE EXTENSION repmgr;
```

### 7️⃣ Register the Primary Node
```bash
repmgr -f /etc/repmgr.conf primary register
```

---

## 💤 Standby Node Setup (`postgresql-b`)

### 1️⃣ Configure `/etc/repmgr.conf`
_(Adjust node_id, node_name, conninfo, location)_
```ini
node_id=2
node_name=postgresql-b
conninfo='host=postgresql-b user=repmgr dbname=userdb-repmgr connect_timeout=2'
data_directory='/pg/data01/'
pg_basebackup_options='--waldir /pg/wal'
pg_bindir='/usr/pgsql-17/bin'
failover=automatic
reconnect_attempts=2
reconnect_interval=1
location='dc2'
promote_command='/usr/pgsql-17/bin/repmgr -f /etc/repmgr.conf standby promote --log-to-file'
follow_command='/usr/pgsql-17/bin/repmgr -f /etc/repmgr.conf standby follow --log-to-file --upstream-node-id=2'
log_file='/pg/log/repmgr.log'
service_start_command   = 'sudo systemctl start postgresql-17.service'
service_stop_command    = 'sudo systemctl stop postgresql-17.service'
service_restart_command = 'sudo systemctl restart postgresql-17.service'
service_reload_command  = 'sudo systemctl reload postgresql-17.service'
```

### 2️⃣ Edit `postgresql.conf`
_(Same as primary node)_

### 3️⃣ Restart PostgreSQL
```bash
sudo systemctl restart postgresql-17.service
```

### 4️⃣ Register the REPMGR Extension
```bash
psql
CREATE EXTENSION repmgr;
```

### 5️⃣ Stop PostgreSQL
```bash
sudo systemctl stop postgresql-17.service
```

### 6️⃣ Clear WAL Directory
```bash
rm -rf /pg/wal/*
```

### 7️⃣ Clone the Standby
```bash
repmgr -f /etc/repmgr.conf standby clone -h postgresql-a -U repmgr -d userdb-repmgr --copy-external-config-files --force
```

### 8️⃣ Start PostgreSQL
```bash
sudo systemctl start postgresql-17.service
```

### 9️⃣ Register the Standby Node
```bash
repmgr -f /etc/repmgr.conf standby register
```

### 🔟 Verify Cluster Status
```bash
repmgr cluster show
```

---

## 👁️ Witness (Quorum) Node Setup (`postgresql-q`)

### 1️⃣ Configure `/etc/repmgr.conf`
_(Adjust node_id, node_name, conninfo, location)_
```ini
node_id=3
node_name=postgresql-q
conninfo='host=postgresql-q user=repmgr dbname=userdb-repmgr connect_timeout=2'
data_directory='/pg/data01/'
pg_basebackup_options='--waldir /pg/wal'
pg_bindir='/usr/pgsql-17/bin'
failover=automatic
reconnect_attempts=2
reconnect_interval=1
location='dc3'
promote_command='/usr/pgsql-17/bin/repmgr -f /etc/repmgr.conf standby promote --log-to-file'
follow_command='/usr/pgsql-17/bin/repmgr -f /etc/repmgr.conf standby follow --log-to-file --upstream-node-id=2'
log_file='/pg/log/repmgr.log'
service_start_command   = 'sudo systemctl start postgresql-17.service'
service_stop_command    = 'sudo systemctl stop postgresql-17.service'
service_restart_command = 'sudo systemctl restart postgresql-17.service'
service_reload_command  = 'sudo systemctl reload postgresql-17.service'
```

### 2️⃣ Edit `postgresql.conf`
_(Same as primary node)_

### 3️⃣ Restart PostgreSQL
```bash
sudo systemctl restart postgresql-17.service
```

### 4️⃣ Register the REPMGR Extension
```bash
psql
CREATE EXTENSION repmgr;
```

### 5️⃣ Create the REPMGR Database
```bash
createdb userdb-repmgr --owner=repmgr
```

### 6️⃣ Register the Extension in the Database
```bash
psql
\c userdb-repmgr
CREATE EXTENSION repmgr;
```

### 7️⃣ Register the Witness Node
```bash
repmgr witness register -h postgresql-a
```

---

## ✅ Verification

Check the cluster status on any node:
```bash
repmgr cluster show
```
You should see something like:

```
 ID | Name         | Role    | Status    | Upstream      | Location | Priority | Timeline | Connection string
----+--------------+---------+-----------+--------------+----------+----------+----------+---------------------------------------------------------------
 1  | postgresql-a | primary | * running |              | dc1      | 100      | 1        | host=postgresql-a user=repmgr dbname=userdb-repmgr connect_timeout=2
 2  | postgresql-b | standby |   running | postgresql-a | dc2      | 100      | 1        | host=postgresql-b user=repmgr dbname=userdb-repmgr connect_timeout=2
 3  | postgresql-q | witness | * running | postgresql-a | dc3      | 0        | n/a      | host=postgresql-q user=repmgr dbname=userdb-repmgr connect_timeout=2
```

---

## 🛠️ Troubleshooting & Tips

- 🔗 Ensure all nodes can communicate over the network.
- 🔥 Check firewall and SELinux settings if connections fail.
- 📜 Use `journalctl -u postgresql-17` for PostgreSQL logs.
- 🆘 Use `repmgr --help` for command options.

---

**🎉 You now have a basic PostgreSQL 17 HA cluster with REPMGR! 🎉**