# FreeBeer24
Step-by-step guide how to reproduce the NiFi demo presented at [FreeBeer24](http://www.freebeersessions.co.za/event/16-november-2017-joburg)

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



---
### For the Twitter -> Slack demo:
* Import the [TwitterToSlack.xml](https://raw.githubusercontent.com/willie-engelbrecht/FreeBeer24/master/templates/TwitterToSlack.xml) template in NiFi:

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/UploadTemplateButton.jpg?raw=true)

* Then click on the template button on the top menu:

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/CreateNewProcessGroupFromTemplate.jpg?raw=true)

* Select which template to create a new process group from:

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/SelectWhichTemplateToCreateFrom.JPG?raw=true)

* Go to: https://apps.twitter.com/  and register an app to get to your secret keys and access token
* Register on Slack: https://slack.com/  and install the "incoming-webhook" app. This will give you a "webhook-URL" that you can use in NiFi

* Update your twitter processor to have the new keys and access token:
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/TwitterSetKeySecret.jpg?raw=true)

* Update your slack processor to have the new webhook-URL:
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/SlackConfigureWebhookURL.jpg?raw=true)

* Start all processors: Press Ctrl+a to select all components, then click on the "play" triangle: 
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/HitPlayButtonToStart.jpg?raw=true)

* You'll know it's working when you can see some flows coming through:

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/TwitterToSlackFlow.JPG?raw=true)



---
### For the TailFile -> Kibana demo:
* Import the [TailFileSSH.xml](https://raw.githubusercontent.com/willie-engelbrecht/FreeBeer24/master/templates/TailFileSSH.xml) template in NiFi:

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/SelectTailFaileSSHTemplate.JPG?raw=true)

* Download the Geo2Lite City database:
```bash
wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz
tar xzvf GeoLite2-City.tar.gz -C /opt
```

* Update the GeoEnrichIP processor, and point the MaxMind Database File property to:
```bash
/opt/GeoLite2-City_20171003/GeoLite2-City.mmdb
```
Example:
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/ConfigureGeoEnrichProcessor.JPG?raw=true)

* Create the following in Elasticsearch:
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

* Start all processors:
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/AllProcessorsRunningForTailFileSSh.JPG?raw=true)

* Browse to your Kibana instance: http://<ip.of.your.machine>:5601

* Click on the "Management" option, then "Index Patterns", then "Create Index Pattern":
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/KibanaManagementIndexPatterns.JPG?raw=true)

* Complete the textbox as "important_* ", let the screen autocomplete, then "Create"

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/KibanaCreateNewIndexPattern.JPG?raw=true)

* On the menu, click "Visualize", then "Create a visualization", then "Coordinate Map":

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/KibanaNewVisualization.JPG?raw=true)

* Select your existing index (in this case, "important_* "):

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/KibanaSelectExistingIndexToVisualize.JPG?raw=true)

* Click on Geohash and set your Field to "location", then click the triangle/play button. You should see:
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/KibanaFinalVisualizationStep.JPG?raw=true)



---
### For the Bitcoin demo:
Import the [TailFileSSH.xml](https://raw.githubusercontent.com/willie-engelbrecht/FreeBeer24/master/templates/TailFileSSH.xml) template in NiFi:

![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/SelectBitcoinTemplate.JPG?raw=true)

* Set your SSL context in the GetHTTP processor (configured earlier, or you can configure it from here):
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/EnableSSLContext.JPG?raw=true)

* Create the following in Elasticsearch
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

* Start all processors:
![](https://github.com/willie-engelbrecht/FreeBeer24/blob/master/images/BitcoinFlow.JPG?raw=true)

* Browse to your Grafana instance:
http://<ip.of.your.machine>:3000/login
Username: admin
Password: admin

# Create a DataSource:



---
### Other commands
* Elasticsearch, view all indexes:
```bash
curl -s -XGET 'http://localhost:9200/_cat/indices?v'
```

* Elasticsearch, delete an index:
```bash
curl -s -XDELETE 'http://localhost:9200/important_locations'
```

* Elasticsearch, view last inserted value:
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
