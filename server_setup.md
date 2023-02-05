# Server Setup
CHEWIE - Cyber Harbinger Enterprise Warbinger Integrated Environment

## Hardware

## ISO 
The server is built using the Security Onion ISO.

1. Download the ISO

### Option 1: Create the bootable USB with Rufus
1.  TODO - Add Instructions


### Option 2: Create the bootable USB with Linux Terminal
1. Verify the USB partition with `lsblk`
- For example, the USB drive may be mounted at `/sdb/sdb1`

2. Ensure the USB is not mounted:
`umount /deb/sdb1`

3. Write and partion with `dd`
```bash
# Format
sudo dd bs=4M if=[location/to/iso/file] of=[partition without the number] conv=fdatasync

# Example
sudo dd bs=4M if=/home/mdtoperator/securityonion.iso of=/dev/sdb conv=fdatasync
```

## RAID Configuration
1. To enter into the RAID controller, on boot press `CTRL` + `r`

2. Delete the current RAID configuration (if necessary)
Press `F2`, "Erase VD" -> "Simple"
- Note: This can take approximately 30-40 minutes

3. Create a new VD
For the Operating System with two physical disks, use RAID 1
For the Operating System with more than two physical disks, use RAID 10
