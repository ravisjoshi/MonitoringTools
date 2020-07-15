## Splunk

### Installation

###


### How to send a server's app logs (e.g. nginx) to Splunk - simplest setup

#### Setup receiver:
    1. Access Splunk GUI (http://192.168.61.101:8000)
    2. Settings --> Forwarding and receiving
    3. Receive data --> Configure receiving --> Add new --> Listen on this port --> 9997

#### Setup Splunk universal forwarder in server where nginx is setup
    1. Download Splunk universal forwarder - in our case `splunkforwarder-8.0.5-a1a6394cc5ae-linux-2.6-x86_64.rpm`
    2. Install `rpm -i splunkforwarder-8.0.5-a1a6394cc5ae-linux-2.6-x86_64.rpm`
    3. `cd /opt/splunkforwarder/bin`
    4. `splunk start --accept-license`
    5. `splunk enable boot-start`
    6. `splunk add forward-server 192.168.61.101:9997 -auth <fwuser>:<fwpassword>`
    7. `splunk add monitor /var/log/nginx/`
    8. Now you can see logs in Splunk
