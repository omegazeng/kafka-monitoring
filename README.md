# Welcome to the kafka-monitoring (JMX) wiki!

## First you have to install and config zabbix-java-gateway
[zabbix java gateway](https://www.zabbix.com/documentation/3.2/manual/concepts/java)

    apt install zabbix-java-gateway

Zabbix server config file (/etc/zabbix/zabbix_server.conf) add:

    JavaGateway=localhost   #zabbix-java-gateway-ip
    JavaGatewayPort=10052
    StartJavaPollers=5

Restart Zabbix server

    service zabbix-java-gateway restart
    service zabbix-server restart

*Note: Check the JavaGatewayPort is listening.*

## Kafka configuration
[How to enable Kafka JMX](https://stackoverflow.com/questions/36708384/enable-jmx-on-kafka-brokers)

A simple way to enable JMX is set JMX_PORT variable, then restart Kafka

**eg.:**
```
# Ubuntu systemd file at /etc/systemd/system/kafka.service
[Unit]
Description=kafka service
Documentation=https://kafka.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target zookeeper.service

[Service]
Type=simple
User=kafka
Group=kafka
Environment=JAVA_HOME=/usr/lib/jvm/java-8-oracle/
Environment=KAFKA_HEAP_OPTS="-Xms4673m -Xmx4673m"
Environment=JMX_PORT="9093"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh
Restart=on-failure
RestartSec=3s
WorkingDirectory=/data/kafka/

[Install]
WantedBy=multi-user.target
```
Restart Kafka server

    systemctl daemon-reload
    service kafka restart

*Note: Check the JMX_PORT is listening.*

# Zabbix configuration
## Upload scripts for discovery JMX

     git clone https://github.com/omegazeng/kafka-monitoring.git 
     cd kafka-monitoring/
     cp jmx_discovery /etc/zabbix/externalscripts
     cp JMXDiscovery-0.0.1.jar /etc/zabbix/externalscripts

## Configuring JMX interfaces
**Click Configuration->Hosts->Kafka Hosts->JMX interfaces->Add**

Enter Kafka IP address and JMX port

## Import template
Download template [zbx_kafka_templates.xml](https://github.com/omegazeng/kafka-monitoring/blob/master/zbx_kafka_templates.xml)

Log in to your zabbix web
**Click Configuration->Templates->Import**

Link Kafka host to this template or add this template to Kafka hosts group.

*Note: You can set JMX_USER and JMX_PASS in Template Macros.*

# Troubles 
If you have problems you can check JMX using this script

[zabbix_get_jmx_3.4.7.sh](https://support.zabbix.com/secure/attachment/59538/zabbix_get_jmx_3.4.7.sh)

**eg.:** bash zabbix_get_jmx_3.4.7.sh  zabbix-java-gateway-ip 10052 server-test-ip 9093
'jmx[java.lang:type=Runtime,Uptime]'

    bash zabbix_get_jmx_3.4.7.sh localhost 10052 ip-x-x-x-x 9093 'jmx[java.lang:type=Runtime,Uptime]'
    {"data":[{"value":"1036057113"}],"response":"success"}
