# FreeBeer24
Step-by-step guide how to reproduce the NiFi demo presented at FreeBeer24

## Prerequisites:
* Perform the following commands as the root user, or use sudo. 
* The steps below requires a RHEL 7 or CentOS 7 system. 
* Make sure you have java installed:
```bash
[root@nifi-demo-0 ~]# java -version
openjdk version "1.8.0_131"
OpenJDK Runtime Environment (build 1.8.0_131-b12)
OpenJDK 64-Bit Server VM (build 25.131-b12, mixed mode)
```

##### Download and install Elasticsearch: https://www.elastic.co/downloads/elasticsearch
```bash
yum -y install https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.3.rpm
systemctl enable elasticsearch
systemctl start elasticsearch
```

##### Download and install Kibana:
```bash
yum -y install https://artifacts.elastic.co/downloads/kibana/kibana-5.6.3-x86_64.rpm
# Update the config file so that Kibana listens on all interfaces
sed -i 's/#server.host: \"localhost\"/server.host: \"0.0.0.0\"/' /etc/kibana/kibana.yml
systemctl enable kibana
systemctl start kibana
```

Browse to: http://<ip.of.your.machine>:5601

##### Download and install Grafana: https://grafana.com/grafana/download
```bash
yum -y install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.6.0-1.x86_64.rpm
systemctl enable grafana-server
systemctl start grafana-server
```

Browse to: http://<ip.of.your.machine>:3000/login
* Username: admin
* Password: admin

##### Download and install Apache NiFi: https://nifi.apache.org/download.html
```bash
wget http://apache.is.co.za/nifi/1.4.0/nifi-1.4.0-bin.tar.gz
tar xzvf nifi-1.4.0-bin.tar.gz -C /opt
/opt/nifi-1.4.0/bin/nifi.sh start
```

Wait a minute, then browse to http://<ip.of.your.machine>:8080/nifi

### In NiFi, create a SSL context, to be used later:
* From your GetHTTP processor, you click on "SSL Context Service":
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/ConfigureSSLContextFromProcessor.JPG?raw=true)
* Truststore password: changeit
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/ConfigureSSLContext.JPG?raw=true)
* Truststore Filename: /etc/pki/java/cacerts
* Truststore Type: JKS
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/ConfigureSSLContextSummary.JPG?raw=true)
* Click Apply, then click on the thunderbolt icon to enable
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/EnableSSLContext.JPG?raw=true)


### For the Twitter -> Slack demo:
* Import the TwitterToSlack.xml template in NiFi
<screenshot here>

* Go to: https://apps.twitter.com/  and register an app to get to your secret keys and access token
* Register on Slack: https://slack.com/  and install the "incoming-webhook" app. This will give you a "webhook-URL" that you can use in NiFi

* Update your twitter processor to have the new keys and access token:
<screenshot here>

* Update your slack processor to have the new webhook-URL:
<screenshot here>

* Start all processors.
<screenshot here>


# For the TailFile -> Kibana demo:
# Import the TailFileSSH.xml template in NiFi

# Download the Geo2Lite City database:
wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz
tar xzvf GeoLite2-City.tar.gz -C /opt

# Update the GeoEnrichIP processor, and point the MaxMind Database File property to: /opt/GeoLite2-City_20171003/GeoLite2-City.mmdb
<screenshot here>

# Create the following in Elasticsearch:
```bash
curl -s -XPUT http://localhost:9200/important_locations -d '
{
  "mappings": {
    "location": {
      "properties": {
        "name": {"type": "string"},
        "location": {"type": "geo_point"}
      }
    }
  }
}' | python -m json.tool
```

# Start all processors.
<screenshot here>


# For the Bitcoin demo:
# Import the Bitcoin.xml template in NiFi
# Set your SSL context in the GetHTTP processor
<screenshot here>

# Create the following in Elasticsearch
```bash
curl -s -XPUT http://localhost:9200/_template/bitstamp_template_1?pretty -d '
{
        "template" : "bitstamp-*",
        "settings": {
                "number_of_shards" : 5,
                "number_of_replicas" : 0
        },
        "mappings" : {
                "_default_" : {
                        "properties" : {
                                "timestamp" : {
                                        "type" : "date",
                                        "format" : "epoch_second"
                                }
                        }
                },
                "bitcoinquotes": {
                        "properties" : {
                                "last_trade" : {
                                        "type" : "scaled_float",
                                        "scaling_factor" : 100,
                                        "index" : false,
                                        "coerce" : true
                                }
                        }
                }
        }
}' | python -m json.tool
```

# Start all processors.
<screenshot here>

# Browse to your Grafana instance:
http://<ip.of.your.machine>:3000/login
Username: admin
Password: admin

# Create a DataSource:




###################### Other commands #########################
# Elasticsearch, view all indexes:
```bash
curl -s -XGET 'http://localhost:9200/_cat/indices?v'
```

# Elasticsearch, delete an index:
```bash
curl -s -XDELETE 'http://localhost:9200/important_locations'
```

# Elasticsearch, view last inserted value:
```bash
curl -s -XGET 'localhost:9200/bitstamp-2017-10-30/_search' -d ' 
{
  "query": {
    "match_all": {}
  },
  "size": 1,
  "sort": [
    {
      "timestamp": {
        "order": "desc"
      }
    }
  ]
}'  | python -m json.tool
```
