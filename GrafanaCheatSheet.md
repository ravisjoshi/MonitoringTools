## Grafana Cheat Sheet


### How to add Zabbix as Datasource in Grafana (not available by default):
* Install the application:

    `grafana-cli plugins install alexanderzobnin-zabbix-app`

* Enable it
    * Log into your Grafana instance
    * Navigate to the `Plugins` section
    * Click the Apps tabs in the Plugins section and select the newly installed app.
    * To enable the app, click the Config tab. Follow the instructions provided with the application and click `Enable`
