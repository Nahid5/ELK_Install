# Docker ELK

Install docker with:
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo 'deb https://download.docker.com/linux/debian stretch stable' > /etc/apt/sources.list.d/docker.list
sudo apt-get update
apt-get remove docker docker-engine docker.io
apt-get install docker-ce
docker run hello-world
```

Get the latest ELK with:
```
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.5.4
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.5.4
```
