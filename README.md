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

    ```
    sudo passwd
    ```

2. Login as root

    ```
    su -
    ```

3. Add source pimox7 + key & update & install rpi-kernel-headers

    ```
    printf "# PiMox7 Development Repo
    deb https://raw.githubusercontent.com/AlNedorezov/pimox7/master/ dev/ \n" > /etc/apt/sources.list.d/pimox.list
    ```

    ```
    curl https://raw.githubusercontent.com/AlNedorezov/pimox7/master/KEY.gpg |  apt-key add -
    ```
    
    ```
    apt update && apt upgrade -y
    ```

4. Make sure that your ip can be resolved to hostname

    ```
    cat /etc/hosts
    ```
    
    prints
    
    ```
    127.0.0.1 localhost
    192.168.1.112 PVE_HOSTNAME
    ```
    
    where 192.168.1.112 is the hostname ip address

5. Install pve-manager separately, and without recommended packages, to avoid packaging issue later

    ```
    DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends -o Dpkg::Options::="--force-confdef" pve-manager
    ```

6. Continue with installation of remaining packages

    ```
    DEBIAN_FRONTEND=noninteractive apt install -y -o Dpkg::Options::="--force-confdef" proxmox-ve
    ```

7. Configure network

    ```
    printf "auto lo
    iface lo inet loopback
    
    iface eth0 inet manual
    
    auto vmbr0
    iface vmbr0 inet static
            address $HOSTNAME_IP_ADDRESS/24
            gateway $GATEWAY_IP_ADDRESS
            bridge-ports eth0
            bridge-stp off
            bridge-fd 0 \n" > /etc/network/interfaces.new
    ```
    where e.g. `$HOSTNAME_IP_ADDRESS`=192.168.1.112, and `$GATEWAY_IP_ADDRESS`=192.168.1.1

8. Reboot

    ```
    reboot
    ```

    after reboot the PVE web interface will be reachable at https://$HOSTNAME_IP_ADDRESS:8006/

### Example VM configuration

```
agent: 1
bios: ovmf
boot: order=scsi0;scsi2;net0
cores: 1
efidisk0: local:100/base-100-disk-1.qcow2,efitype=4m,pre-enrolled-keys=1,size=64M
memory: 2048
meta: creation-qemu=7.0.0,ctime=1699409858
name: vm100
net0: virtio=4A:AE:C5:F6:00:9C,bridge=vmbr0,firewall=1
numa: 0
ostype: other
scsi0: local:100/base-100-disk-0.qcow2,size=32G
scsi2: local:iso/ubuntu-22.04.3-live-server-arm64.iso,media=cdrom,size=2021436K
scsihw: virtio-scsi-pci
smbios1: uuid=dedc20f5-b9f3-4b21-a9ad-276668451a20
sockets: 1
template: 1
#qmdump#map:efidisk0:drive-efidisk0:local:qcow2:
#qmdump#map:scsi0:drive-scsi0:local:qcow2:
```

Notes
---
1. This repo just contains the precompiled debian packages. The original Proxmox sources can be found at https://git.proxmox.com
2. The (very minimally) patched sources to rebuild this can be found at https://github.com/pimox
