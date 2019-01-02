# Install instructions for ELK

## Install Java 8 and Elastic repo
```
sudo apt install software-properties-common apt-transport-https -y
sudo add-apt-repository ppa:webupd8team/java -y
sudo apt-get update
sudo apt-get -y install oracle-java8-installer
```
https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html

Make sure java 8 is confured to use with:
```
update-alternatives --config java
```

Also add java.sh profile with:
```
vim /etc/profile.d/java.sh
```

And paste the following:
```
#Set JAVA_HOME
JAVA_HOME="/usr/lib/jvm/java-8-oracle"
export JAVA_HOME
PATH=$PATH:$JAVA_HOME
export PATH
```

And set it to execution with:
```
chmod +x /etc/profile.d/java.sh
source /etc/profile.d/java.sh
```

Add the elastic repo with:
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```





## Install Elasticsearch
```
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
Restart elasticsearch and have it run at startup with:
```
systemctl start elasticsearch
systemctl enable elasticsearch
```

You can test with:
```
curl -XGET 'localhost:9200/?pretty'
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
server.host: "localhost"
elasticsearch.url: "http://localhost:9200"
```
Now enable the Kibana service, and start it:
```
sudo systemctl enable kibana
sudo systemctl start kibana
```






## Install Nginx - Required to access Kibana from external hosts
```
sudo apt install nginx apache2-utils -y
sudo htpasswd -c /etc/nginx/htpasswd.users <SOME_USER_NAME>
```
Add another config:
```
vim /etc/nginx/sites-available/kibana
```
And add this in it:
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

Activate the kibana virtual host and test all configs with:
```
ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/
nginx -t
```
Restart and start nginx at startup with:
```
systemctl enable nginx
systemctl restart nginx
```



## Generate SSL Certificates
```
sudo mkdir -p /etc/pki/tls/certs
sudo mkdir /etc/pki/tls/private
```
If you dont have DNS setup:
```
sudo vim /etc/ssl/openssl.cnf
```
Find `[ v3_ca ]` and add the following under the line:
```
subjectAltName = IP: ELK_server_private_IP
```

Save and generate the SSLk certificate and private key with:
```
cd /etc/pki/tls
sudo openssl req -config /etc/ssl/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt
```

If you have FQDN (DNS)
```
cd /etc/pki/tls; sudo openssl req -subj '/CN=ENTER_ELK_FQDNS_HERE/' -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt
```
The logstash-forwarder.crt will be distributed to all of the servers.




## Install Logstash
```
sudo apt-get install logstash
```

Then in you server's /etc/logstash/conf.d/
add the configuration for input, filter, and output
The following is a template for the config (Note you can have differing files for the input, filter, and output)
```
# The # character at the beginning of a line indicates a comment. Use
# comments to describe your configuration.
input {
}
# The filter part of this file is commented out to indicate that it is
# optional.
# filter {
#
# }
output {
}
```
After you have your logstash settings restart and have it start at startup
```
sudo systemctl enable logstash
sudo systemctl start logstash
```
