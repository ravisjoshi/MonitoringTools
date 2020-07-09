## Zabbix

### Installation:
https://www.zabbix.com/download

#### Install Zabbix repository
```
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/8/x86_64/zabbix-release-5.0-1.el8.noarch.rpm
dnf clean all
```
#### Install Zabbix server, frontend, agent
`dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-agent`

#### Create initial database
Run the following on your database host.
```
mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

On Zabbix server host import initial schema and data. You will be prompted to enter your newly created password.
```
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

#### Configure the database for Zabbix server

Edit file `/etc/zabbix/zabbix_server.conf`
```
DBPassword=password
```
#### Configure PHP for Zabbix frontend
Edit file `/etc/php-fpm.d/zabbix.conf`, uncomment and set the right timezone for you.
```
; php_value[date.timezone] = Europe/Riga
```
### Start Zabbix server and agent processes

#### Disable selinux
```
getenforce
setenforce 0
```
Edit file `/etc/selinux/config` for permanent changes, requires reboot:
```
SELINUX=permissive
```

#### Start Zabbix server and agent processes and make it start at system boot.
```systemctl restart zabbix-server zabbix-agent httpd php-fpm
systemctl enable zabbix-server zabbix-agent httpd php-fpm
```
#### Configure Zabbix frontend
Connect to your newly installed Zabbix frontend: http://server_ip_or_name/zabbix

**NOTE:-** If you are unable to access the zabbix UI(issue faced in VBox VM) check below:
```
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```
----
----
### Agent installation on host to monitor
* Switch to non-root user:
```
mkdir zabbix-agent && cd zabbix-agent
wget https://www.zabbix.com/downloads/5.0.1/zabbix_agent-5.0.1-linux-3.0-amd64-static.tar.gz
tar -xzf zabbix_agent*.tar.gz
vi conf/zabbix_agentd.conf
Server=<zabbixMonitoringServerIP>
```
**NOTE:** If you are running it with root user update AllowRoot parameter in `conf/zabbix_agentd.conf` too:
```
AllowRoot=1
```

Start the process:
`./sbin/zabbix_agentd -c conf/zabbix_agentd.conf`
