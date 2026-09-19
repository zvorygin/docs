# Installation of amnezia-wg on Raspberry Pi

## Introduction
Amnezia-wg is wireguard-based protocol, that adds random noise at the handshake phase, to confuse firewalls and allow
the connection. 

Thus there are two corollaries:
1. Both client and server have to agree on noise parameters, to be able to have successful handshake
1. Most of the commands/flags applicable to wireguard are applicable to amnezia-wg as well

I'm using a dedicated Raspberry Pi 5 for this project, so don't have to use docker containers or manage some other
processes.

I'm going to have site-to-site setup, and use Raspberry Pi on two locations, to estabilish the transparent network
peering. Each site would have dedicated network, that would have said Raspberry Pi installed there.

Networks on SiteA and SiteB would have format 192.168.A.0/24 and 192.168.B.0/24 accordingly. Each network would have
a standard 192.168.A.1 and 192.168.B.1 gateway, so that Raspberry Pi would have access to the internet to act as a VPN
server, and also would have access to remote site (with proper port forwarding configured on static IP), to act as 
VPN client. Raspberry Pi-es on sites would have static address "192.168.X.2", that DHCP server (at 192.168.X.1) will
advertise everyone to use as a gateway. Raspberry Pi, for obvious reasons, to avoid cyclic loops, would use gateway
192.168.X.1.

## Raspberry Pi flashing

Use Raspberry Pi imager available at [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/).
I use Raspberry Pi OS Lite to minimize the footpring, and SSH key-based authentication to strenghen the security and
simplify the routines.

```bash
# Update package definitions and upgrade whatever is upgradeable
sudo apt update
sudo apt upgrade
```

## Compiling amnezia-wg from sources

Amnezia-wg comes in two pieces:
1. Kernel module
2. Tools to interact with said kernel module

We have to install both first, and then wire everything together.

```bash
# Install git, vim and resolveconf that we would need later
sudo apt-get install git vim resolvconf

# Reboot, as resolvconf may write empty /etc/resolv.conf and DNS resolution would stop working
sudo reboot now
```

Build kernel modules

```bash
mkdir ~/amnezia
cd ~/amnezia

# Checkout sources 
git clone https://github.com/amnezia-vpn/amneziawg-linux-kernel-module.git

# Build and install kernel module
cd ~/amnezia/amneziawg-linux-kernel-module/src/

# Link kernel sources - we need the one that ends with "+rpt-common-rpi" at /usr/src
ln -s /usr/src/linux-headers-<your Raspberry Pi kernel version>+rpt-common-rpi/ kernel

# Build and install
make
sudo make install

# Verify installation
modprobe -c | grep amnezia
```

Build tooling

```bash

cd ~/amnezia

# Checkout sources
git clone https://github.com/amnezia-vpn/amneziawg-tools

cd ~/amnezia/amneziawg-tools/src/

# Build and install
make
sudo make install
```

Since amnezia-wg is based on original wireguard(and essentially forks of appropriate kernel/tools repositories),
most of wireguard commands/troubleshooting is applicable. The difference, is that instead of `wg` command, `awg`
command is used. Instead of `wg-quick` helper, `awg-quick` helper is used and so on ...