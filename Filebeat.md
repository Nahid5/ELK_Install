# Installing and using Filebeat

Make sure that on the clients you have filebeats from elastic using:
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt update
sudo apt install filebeat -y
```

Next if you are using SSL then you have to copy over the cert from the server to the host :
```
scp /etc/pki/tls/certs/logstash-forwarder.crt user@client_server_private_address:/tmp
```
Then move it to a safe directory:
```
sudo mkdir -p /etc/pki/tls/certs
sudo cp /tmp/logstash-forwarder.crt /etc/pki/tls/certs/
```

Then edit filebeat config on the client using:
```
vim /etc/filebeat/filebeat.yml
```
Set *enabled: true* 
Then define the log files to be sent to the logstash server, and the example below will add ssh log file 'auth.log'
```
paths:
    - /var/log/auth.log
    - /var/log/syslog
```
Then set the output to logstash by commenting out the default 'elasticsearch' output and uncommt the logstash output line as below:
```
output.logstash:
  # The Logstash hosts
  hosts: ["THE_ELK_HOST:5443"]
  ssl.certificate_authorities: ["/etc/pki/tls/certs/logstash-forwarder.crt"]
```

Then enable filebeat modules by editing filebeat refrences
```
vim /etc/filebeat/filebeat.reference.yml
```
and add
```
- module: system
  # Syslog
  syslog:
    enabled: true
```
Then start have have filebeat start at startup using:
```
systemctl start filebeat
systemctl enable filebeat
```
