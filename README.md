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
