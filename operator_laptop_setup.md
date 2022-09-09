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

1. Clone this repo
```bash
cd ~
sudo git clone https://github.com/1dentified/Eldritch_Knowledge.git
```
1. Edit /etc/sudoers and add the following line to the bottom. "mdtoperator ALL=(ALL) NOPASSWD: ALL"

2. Update and Upgrade!
```bash
sudo apt update && sudo apt upgrade
```

3. APT
```bash
sudo apt -y install docker.io docker-compose wireshark tmux vim curl net-tools python3-pip
```

4. Snap
```bash
sudo snap install --classic code
sudo snap install --classic powershell
```


## Elastic (Elasticsearch and Kibana)

1. Configure mem settings
```bash
sudo vim /etc/sysctl.conf

# Set VM max - add the following line:
vm.max_map_count=262144

# Reload configuration
sudo sysctl -p
```

2. Setup Docker and pull down the containers
```bash
cd /opt/

sudo docker network create elastic
sudo docker pull docker.elastic.co/elasticsearch/elasticsearch:8.2.0
sudo docker pull docker.elastic.co/kibana/kibana:8.2.0
```

3. Start Elasticsearch
```bash
sudo docker run -d --name es01 --net elastic -p 9200:9200 -it docker.elastic.co/elasticsearch/elasticsearch:8.2.0
```

4. Log into the container, set the password, and create an enrollmentkey
```bash
sudo docker exec -it es01 /bin/bash

# now in the container terminal
cd bin
elasticsearch-reset-password -i -u elastic   # Set passwords according to local requirements
elasticsearch-create-enrollment-token -s kibana   # Copy token base64 text and save it for kibana enrollement
exit
```

Important! Make sure you take not of the password and enrollment key!

5. Start Kibana
```bash
sudo docker run --name kibana -d --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.2.0
```

6. Navigate to: `localhost:5601`

7. Paste in the base64 enrollment token you got from step 4.

8. If required, log into the kibana container and obtain a verification code
```bash
sudo docker exec -it kibana /bin/bash

# In the container:
kibana-verification-code
exit
```

9. Login with: `elastic:[pw]`

10. DONE! Wait...not yet...go set dark mode! (Stack Management -> Kibana -> Advanced Settings)


## Suricata
1. Install Suricata
```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install -y suricata
```

2. Update the config file
```bash
sudo vim /etc/suricata/suricata.yaml

# Line 730
mqtt:
  enabled: yes  
# Line 769
rdp:
  enabled: yes
# Line 949
modbus:
  enabled: yes
# Line 980
sip:
  enabled: yes
```

3. Enable and Update signatures (ruleset)
```bash
sudo suricata-update enable-source et/open
sudo suricata-update enable-source oisf/trafficid
sudo suricata-update enable-source sslbl/ssl-fp-blacklist
sudo suricata-update enable-source sslbl/ja3-fingerprints
sudo suricata-update enable-source etnetera/aggressive
sudo suricata-update enable-source tgreen/hunting
sudo suricata-update enable-source malsilo/win-malware

sudo suricata-update
```


## Zeek
```bash
sudo apt -y install cmake make gcc g++ flex bison libpcap-dev libssl-dev python3 python3-dev swig zlib1g-dev
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

2. Generate an Elasticsearch SSL Fingerprint
```bash
sudo docker exec -it es01 openssl x509 -fingerprint -sha256 -in config/certs/http_ca.crt

# Copy the hex output after "SHA256 Fingerprint="

# You need to get rid of the ":" characters
echo "HEX:CODE" | sed s/://g

# Copy the output - you'll need this for the next step!
```

3. Update the configurations:
```bash
sudo vim /etc/filebeat/filebeat.yml    # Or use nano if you need someone to hold your hand!

# Uncomment and change:
## Line 110: Uncommment and update to host: "http://0.0.0.0:5601"
## Line 137: change to hosts: ["https://0.0.0.0:9200"]

## Starting on Line 144 (Spacing is super important!)
  username: "elastic"
  password: "[pw]"
  ssl:
    enabled: true
    ca_trusted_fingerprint: "[HEXCODE]"
```

4. Save changes
  
5. Start filebeat setup:
```bash
sudo filebeat setup -e
```

6. Enable the Zeek, Suricata, and System modules:
```bash
sudo filebeat modules enable zeek 
sudo filebeat modules enable suricata
```

7. Replace zeek.yml with the zeek.yml from this repo.
```
sudo cp -f /home/mdtoperator/Eldritch_Knowledge/zeek.yml /etc/filebeat/modules.d/
```

8. Do the same with the suricata yml file.
```
sudo cp -f /home/mdtoperator/Eldritch_Knowledge/suricata.yml /etc/filebeat/modules.d/
```

9. Run filebeat setup.
```
sudo filebeat setup -e
```


## Firefox Configuration
1. Open firefox
2. Open the settings dropdown and select "settings"
3. Scroll down and choose "Dark Theme" or I will judge you.
4. On the left, choose Home.
5. Change the drop down from firefox Default to Custom Urls...
6. Copy the following location into the url field: file:///home/mdtoperator/Eldritch_Knowledge/necronomicron.html
7. There is no save, simply open a new tab and click the home button in the top left to test!
