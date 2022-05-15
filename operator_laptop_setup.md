# Operator Laptop Setup

## Base OS: Ubuntu 20.04

> We don't go with the most up to date long term relase becasue the current 22.04 doesn't not yet have enough mileage to test the new kernel, drivers and pacakages for compatibilty.

1. Download Ubuntu ISO for 22.04 LTS
https://releases.ubuntu.com/20.04/

2. User Rufus to create a bootable USB with the ISO.
https://rufus.ie/en/

3. Boot to the USB and follow instructions for installation. Make sure to wipe the current drives.
> If possible use a smaller faster solid state drive for the OS and a slower "spinner" hard drive for data storage.

4. Install Docker.io
```
sudo apt install docker.io
```
5. Install Docker-Compose.

6. Install vscode
`sudo snap install --classic code`



7. Setup Elastic Nodes
  - `cd /opt`
  -   Set vm max `sudo sysctl -w vm.max_map_count=262144`
  - `sudo docker network create elastic`
  - `sudo docker pull docker.elastic.co/elasticsearch/elasticsearch:8.2.0`
  - `sudo docker pull docker.elastic.co/kibana/kibana:8.2.0`
  - ` sudo docker run -d --name es01 --net elastic -p 9200:9200 -it docker.elastic.co/elasticsearch/elasticsearch:8.2.0`
  - In the out put you need to find two items.
    - Set elastic Password
      - `sudo docker exec -it es01 /bin/bash`
         - now in the container terminal:
           - `cd bin`
           - `elasticsearch-reset-password -i -u elastic`
            > set passworda according to local requirements
            - `eleasticsearch-create-enrollmenttken -s kibana
            > copy token base64 text and save it for kiban enrollement
            - `exit`
    - Start kibana container
    `sudo docker run --name kibana -d --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.2.0`
    - Wait fro the generated link and ctrl+click or copy the link int the browser.
    - Click connect to elasticsearh.
    - Copy enrollment token and click connect.
    - Login with elastic:<pw>
    - DONE! Wait...not yet...go set dark mode.

## Suricata
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install -y suricata
- Update suricata
  

## Zeek
sudo sudo apt-get -y install cmake make gcc g++ flex bison libpcap-dev libssl-dev python3 python3-dev swig zlib1g-dev
sudo git clone --recursive https://github.com/zeek/zeek
sudo chmod -R 755 zeek
sudo chown -R mdtoperator:mdtoperator zeek
cd zeek
./configure && make && sudo make install

 ## Filebeat
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.2.0-amd64.deb
sudo dpkg -i filebeat-oss-8.2.0-amd64.deb
  
  
 ## Wireshark
sudo apt install wireshark

 ## Powershell
 sudo snap install --classic powershell
