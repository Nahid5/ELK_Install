# Install instructions for ELK

## Install Java 8
```
sudo add-apt-repository -y ppa:webupd8team/java
sudo apt-get update
sudo apt-get -y install oracle-java8-installer
```
https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html


## Install Elasticsearch
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update && sudo apt-get install elasticsearch
```

### Configure Elasticsearch
```
sudo vim /etc/elasticsearch/elasticsearch.yml
```
Set the networkhost to localhost
```
network.host: localhost
http.port:9200
```
Restart elasticsearch
```
sudo service elasticsearch restart
```
Then run the following command to start Elasticsearch on boot up:
```
sudo update-rc.d elasticsearch defaults 95 10
```

## Install Kibana
```
sudo apt-get install kibana
```
### Configure Kibana
```
sudo vim /etc/kibana/kibana.yml
```
And set
```
server.port: 5601
elasticsearch.url: "http://localhost:9200"
```
Now enable the Kibana service, and start it:
```
sudo update-rc.d kibana defaults 96 9
sudo service kibana start
```

# Install Nginx - Required to access Kibana from external hosts
```
sudo apt-get install nginx apache2-utils
sudo htpasswd -c /etc/nginx/htpasswd.users <SOME_USER_NAME>
```
Edit the config:
```
sudo vim /etc/nginx/sites-available/default
```
Replace everythign inside with this:
```
server {
	listen 80;

	server_name example.com;

	auth_basic "Restricted Access";
	auth_basic_user_file /etc/nginx/htpasswd.users;

	location / {
		proxy_pass http://localhost:5601;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;        
	}
}
```
Restart the Nginx server:
```
sudo service nginx restart
```

## Install Logstash
```
sudo apt-get install logstash
```
Test logstash config using:
```
sudo service logstash configtest
```

## Setup rsyslog
On ELK server:
```
sudo apt-get install rsyslog
```

Change config on rsyslog.config, and uncomment the lines
```
# provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 514

# provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
```

```
sudo vim /etc/rsyslog.d/01-json-template.conf
```
And add
```
template(name="json-template"
  type="list") {
    constant(value="{")
      constant(value="\"@timestamp\":\"")     property(name="timereported" dateFormat="rfc3339")
      constant(value="\",\"@version\":\"1")
      constant(value="\",\"message\":\"")     property(name="msg" format="json")
      constant(value="\",\"sysloghost\":\"")  property(name="hostname")
      constant(value="\",\"severity\":\"")    property(name="syslogseverity-text")
      constant(value="\",\"facility\":\"")    property(name="syslogfacility-text")
      constant(value="\",\"programname\":\"") property(name="programname")
      constant(value="\",\"procid\":\"")      property(name="procid")
    constant(value="\"}\n")
}
```

```
sudo vim etc/rsyslog.d/60-output.conf
```
And add
```
# This line sends all lines to defined IP address at port 10514,
# using the "json-template" format template

*.*                         @private_ip_logstash:10514;json-template
```

Restart the rsyslog service with:
```
sudo service rsyslog restart
```

```
sudo vim /etc/logstash/conf.d/logstash.conf
```
And add
```
# This input block will listen on port 10514 for logs to come in.
# host should be an IP on the Logstash server.
# codec => "json" indicates that we expect the lines we're receiving to be in JSON format
# type => "rsyslog" is an optional identifier to help identify messaging streams in the pipeline.

input {
  udp {
    host => "logstash_private_ip"
    port => 10514
    codec => "json"
    type => "rsyslog"
  }
}

# This is an empty filter block.  You can later add other filters here to further process
# your log lines

filter { }

# This output block will send all events of type "rsyslog" to Elasticsearch at the configured
# host and port into daily indices of the pattern, "rsyslog-YYYY.MM.DD"

output {
  if [type] == "rsyslog" {
    elasticsearch {
      hosts => [ "elasticsearch_private_ip:9200" ]
    }
  }
}
```

Test the configs using:
```
sudo service logstash configtest
```
Then
```
sudo service logstash restart
sudo service rsyslog restart
```

In the client:
```
sudo vim /etc/rsyslog.d/50-default.conf
```
And add the following config:
```
*.*                         @private_ip_of_ryslog_server:514
```
Restart rsyslog
```
sudo service rsyslog restart
```


