## Splunk

### Installation
**Ref:** https://www.splunk.com/en_us/download.html


###

----
----

## Splunk Universal Forwarder

### Method 1: How to send a server's app logs (e.g. nginx) to Splunk - simplest setup

#### Setup receiver:
1. Access Splunk GUI (http://192.168.61.101:8000)
2. Settings --> Forwarding and receiving
3. Receive data --> Configure receiving --> Add new --> Listen on this port --> 9997

#### Setup Splunk universal forwarder in server where nginx is setup
**Ref:** https://www.splunk.com/en_us/download.html
1. Download Splunk universal forwarder - in our case `splunkforwarder-8.0.5-a1a6394cc5ae-linux-2.6-x86_64.rpm`
2. Install `rpm -i splunkforwarder-8.0.5-a1a6394cc5ae-linux-2.6-x86_64.rpm`
3. `cd /opt/splunkforwarder/bin`
4. Start Splunk: `/opt/splunkforwarder/bin/splunk start --accept-license`
5. Enable boot-start/init script: `/opt/splunkforwarder/bin/splunk enable boot-start`
6. `/opt/splunkforwarder/bin/splunk add forward-server 192.168.61.101:9997 -auth <fwuser>:<fwpassword>`
7. Test Forwarder connection: `/opt/splunkforwarder/bin/splunk list forward-server -auth <fwuser>:<fwpassword>`
8. Add Data: `/opt/splunkforwarder/bin/splunk add monitor /var/log/nginx/ -index main -sourcetype nginx-logs`
9. Now you can see logs in Splunk

----

### Method 2: How to send a server's app logs (e.g. nginx) to Splunk - simplest setup
