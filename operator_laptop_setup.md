# Operator Laptop Setup

## Base OS: Ubuntu 20.04

> We don't go with the most up to date long term release becasue the current 22.04 doesn't not yet have enough mileage to test the new kernel, drivers and pacakages for compatibilty.

1. Download Ubuntu ISO for 22.04 LTS
https://releases.ubuntu.com/20.04/

2. User Rufus to create a bootable USB with the ISO.
https://rufus.ie/en/

3. Boot to the USB and follow instructions for installation. Make sure to wipe the current drives.
> If possible use a smaller faster solid state drive for the OS and a slower "spinner" hard drive for data storage.

## Install Core Packages

1. Docker.io
```
sudo apt install docker.io
```
2. Docker-Compose.
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

3. VScode
`sudo snap install --classic code`

4. Wireshark
```
sudo apt install wireshark
```

5. Powershell
```
sudo snap install --classic powershell
```

## Elastic (Elasticsearch and Kibana)

1. Configure settings
```
cd /opt

# Set VM max
sudo sysctl -w vm.max_map_count=262144
```

2. Setup Docker and pull down the containers
```
sudo docker network create elastic
sudo docker pull docker.elastic.co/elasticsearch/elasticsearch:8.2.0
sudo docker pull docker.elastic.co/kibana/kibana:8.2.0
```

3. Start Elasticsearch
```
sudo docker run -d --name es01 --net elastic -p 9200:9200 -it docker.elastic.co/elasticsearch/elasticsearch:8.2.0
```

4. Log into the container, set the password, and create an enrollmentkey
```
sudo docker exec -it es01 /bin/bash

# now in the container terminal
cd bin
elasticsearch-reset-password -i -u elastic   # Set passwords according to local requirements
eleasticsearch-create-enrollmentkey -s kibana   # Copy token base64 text and save it for kibana enrollement
exit
```

Important! Make sure you take not of the password and enrollment key!

5. Start Kibana
```
sudo docker run --name kibana -d --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.2.0
```

6. Wait for the generated link and ctrl+click or copy the link int the browser.

7. Click connect to Elasticsearh.

8. Copy enrollment token and click connect.

9. Login with: `elastic:[pw]`

10. DONE! Wait...not yet...go set dark mode. (Kibana -> Advanced Settings)


## Suricata
```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install -y suricata
```

TODO: Install Emerging Threats Ruleset Instructions

## Zeek
```bash
sudo sudo apt-get -y install cmake make gcc g++ flex bison libpcap-dev libssl-dev python3 python3-dev swig zlib1g-dev
sudo git clone --recursive https://github.com/zeek/zeek
sudo chmod -R 755 zeek
sudo chown -R mdtoperator:mdtoperator zeek
cd zeek
./configure && make && sudo make install
```

## Filebeat
1. Install Filebeat
```bash
sudo curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.2.0-amd64.deb
sudo dpkg -i filebeat-oss-8.2.0-amd64.deb
```

2. Update the configurations:
```
sudo vim /etc/filebeat/filebeat.yml    # Or use nano if you need someone to hold your hand!

# Uncomment and change:
## Line 110: Uncommment and update to host: "http://0.0.0.0:5601"
## Line 137: change to hosts: ["https://0.0.0.0:9200"]
## Line 144 Uncomment and update username: `elastic`
## Line 145: Uncomment and update password: `[pw]`
```

3. Save chages
  
4. Start filebeat setup:
```
sudo filebeat setup -e
```

5. Enable the Zeek and Suricata modules:
```
sudo filebeat modules enable zeek 
sudo filebeat modules enable suricata
```

TODO: Log file path configuration

