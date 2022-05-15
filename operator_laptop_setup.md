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
sudo apt insteall docker.io
```
5. Install Docker-Compose.
