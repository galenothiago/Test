# Zabbix 3 Install on CEntOS 7 with PostgreSQL 9.5

## Repo installation

### Zabbix repo

```bash
yum install http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
```

### PostgreSQL repo

```bash
yum install https://download.postgresql.org/pub/repos/yum/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm
```

## PostgreSQL Configuration

### Installation

```bash
yum install postgresql95-server postgresql95-contrib
```

### Setup

Create the database cluster:

```bash
/usr/pgsql-9.5/bin/postgresql95-setup initdb
```

Start the database:

```bash
systemctl start postgresql-9.5
```

Adjust the `pg_hba.conf`, allowing local access without password, replacing:

```bash
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```

By:

```bash
# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
```

Then, apply this changes:

```bash
systemctl reload postgresql-9.5
```

Now, apply the server tuning (based on [pgconfig.org](http://pgconfig.org) suggestions): 

* [https://www.pgconfig.org/#/tuning?total_ram=5&max_connections=60&enviroment_name=WEB&pg_version=9.5&share_link=true](https://www.pgconfig.org/#/tuning?total_ram=5&max_connections=60&enviroment_name=WEB&pg_version=9.5&share_link=true).

### Configuration

Create the database and user zabbix:

```sql
CREATE USER zabbix;
CREATE DATABASE zabbix OWNER zabbix;
```

## Zabbix

### Installation

```bash
yum install zabbix-server-pgsql zabbix-web-pgsql zabbix-agent zabbix-get
```

### Server Configuration

Run the database setup:

```bash
cd /usr/share/doc/zabbix-server-pgsql-3.0.4/
zcat create.sql.gz | psql -U zabbix -d zabbix
```

Configure the system auto-start:

```bash
systemctl enable zabbix-server
systemctl enable zabbix-agent
```

Adjust the `/etc/zabbix/zabbix_server.conf`, adding:

```bash
DBUser=zabbix
DBPort=5432
DBHost=localhost
```

Now, start the services:

```bash
systemctl start zabbix-server
systemctl start zabbix-agent
```

### Web interface Configuration

Adjust the `/etc/httpd/conf.d/zabbix.conf`, uncommenting the `date.timezone` configuration:

```bash
php_value date.timezone America/Sao_Paulo
```

Configure the system auto-start:

```bash
systemctl enable httpd
```

Finally, start the server:

```bash
systemctl start httpd
```


## References

1. http://blog.dnslink.com.br/BG/instalando-zabbix-3-no-centos-7/
1. https://www.zabbix.com/documentation/3.0/pt/manual/installation/install_from_packages
1. https://www.zabbix.com/documentation/3.0/pt/manual/appendix/install/db_scripts
