Pimox - Proxmox V7 for the Raspberry Pi
===

Pimox is a port of Proxmox to the Raspberry Pi allowing you to build a Proxmox cluster of Rapberry Pi's or even a hybrid cluster of Pis and x86 hardware.

Requirements
---
* Raspberry Pi 4
* Internet connection via ethernet

Install from "scratch", RPiOS64bit Interactive Automatic Installer
---
1. Flash and startup the latest image from https://downloads.raspberrypi.org/raspios_arm64/ .
2. sudo -s
3. curl https://raw.githubusercontent.com/AlNedorezov/pimox7/master/RPiOS64-IA-Install.sh > RPiOS64-IA-Install.sh
4. chmod +x RPiOS64-IA-Install.sh
5. ./RPiOS64-IA-Install.sh
6. Follow the prompts

Manual installation
---
Prechecks

1. Pre-installed Debian __Bullseye__ based  ___64-bit___ OS (not 32bit)
2. In /etc/network/interfaces, give the Pi a static IP address. You cannot use dhcp.
3. In /etc/network/interfaces, remove any IPv6 addresses.
4. In /etc/hostname, make sure the Pi has a name.
5. In /etc/hosts, make sure this hostname corresponds to the static IP you previous set.
6. Make sure the kernel-headers are installed.

Installation
1. echo "deb https://raw.githubusercontent.com/AlNedorezov/pimox7/master/ dev/" > /etc/apt/sources.list.d/pimox.list
2. curl https://raw.githubusercontent.com/AlNedorezov/pimox7/master/KEY.gpg |  apt-key add -
3. apt update
4. apt install proxmox-ve (use a local attatched console! Network connections will be lost/reset during installation progress)

### If a static IP address configured in dhcp
1. Set a password for root for a web gui login

    `sudo passwd`

2. Login as root

    `su -`

3. Add source pimox7 + key & update & install rpi-kernel-headers

    ```
    printf "# PiMox7 Development Repo
    deb https://raw.githubusercontent.com/AlNedorezov/pimox7/master/ dev/ \n" > /etc/apt/sources.list.d/pimox.list
    ```
    
    `curl https://raw.githubusercontent.com/AlNedorezov/pimox7/master/KEY.gpg |  apt-key add -`
    
    `apt update && apt upgrade -y`

4. Make sure that your ip can be resolved to hostname

    `cat /etc/hosts`
    
    prints
    
    ```
    127.0.0.1 localhost
    192.168.1.112 PVE_HOSTNAME
    ```
    
    where 192.168.1.112 is the hostname ip address

5. Install pve-manager separately, and without recommended packages, to avoid packaging issue later

    `DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends -o Dpkg::Options::="--force-confdef" pve-manager`

6. Continue with installation of remaining packages

    `DEBIAN_FRONTEND=noninteractive apt install -y -o Dpkg::Options::="--force-confdef" proxmox-ve`

7. Reboot

    `reboot`

    after reboot the PVE web interface will be reachable at https://$HOSTNAME_IP_ADDRESS:8006/


Notes
---
1. This repo just contains the precompiled debian packages. The original Proxmox sources can be found at https://git.proxmox.com
2. The (very minimally) patched sources to rebuild this can be found at https://github.com/pimox
