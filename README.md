# Welcome to the kafka-monitoring (JMX) wiki!

## First you have to install and config zabbix-java-gateway
[zabbix java gateway](https://www.zabbix.com/documentation/3.2/manual/concepts/java)

    apt install zabbix-java-gateway

Zabbix server config file (/etc/zabbix/zabbix_server.conf) add:

    JavaGateway=localhost
    JavaGatewayPort=10052
    StartJavaPollers=5

Restart zabbix server

    service zabbix-java-gateway restart
    service zabbix-server restart

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
Restart kafka server

    service kafka restart

# Zabbix configuration

# Upload scripts for discovery JMX

     git clone https://github.com/omegazeng/kafka-monitoring.git 
     cd kafka-monitoring/
     cp jmx_discovery /etc/zabbix/externalscripts
     cp JMXDiscovery-0.0.1.jar /etc/zabbix/externalscripts

## Import template
Log in to your zabbix web

**Click Configuration->Templates->Import**

Download template [zbx_kafka_templates.xml](https://github.com/omegazeng/kafka-monitoring/blob/master/zbx_kafka_templates.xml) and upload to zabbix
Then add this template to Kafka and configure JMX interfaces on zabbix 

Enter Kafka IP address and JMX port
If you see jmx icon, you configured JMX monitoring  good!

# Troubles 
if you have problems you can check JMX using this script
     #!/usr/bin/env bash
     
     ZBXGET="/usr/bin/zabbix_get"
     if [ $# != 5 ]
     then
     echo "Usage: $0 <JAVA_GATEWAY_HOST> <JAVA_GATEWAY_PORT> <JMX_SERVER> <JMX_PORT> <KEY>"
     exit;
     fi
     QUERY="{\"request\": \"java gateway jmx\",\"conn\": \"$3\",\"port\": $4,\"keys\": [\"$5\"]}"
     $ZBXGET -s $1 -p $2 -k "$QUERY"

**eg.:** ./zabb_get_java  zabbix-java-gateway-ip 10052 server-test-ip 12345 
'jmx[java.lang:type=Threading,PeakThreadCount]'

