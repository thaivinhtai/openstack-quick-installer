# OpenStack Stein | Mini LAB

A small script used for deploy OpenStack Stein

Supported OS:
- Ubuntu Bionic Beaver 18.04 LTS
- Ubuntu Eoan 19.04

## Topology

![topo](./images/topo.png)

## Hardware requirements

![requirement_hardware](./images/requirement_hardware.png)

# Step 1: Network setup

Changing network interfaces name

Edit your /etc/default/grub changing the line from `GRUB_CMDLINE_LINUX=""` to `GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"`
and, finally:

```sh
$ sudo update-grub
$ sudo reboot
```

Ubuntu 18.04 moved `/etc/network/interfaces` to netplan. You need update IP config at `/etc/netplan/*.yaml`

Example config `/etc/netplan/01-netcfg.yaml`

```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: yes
    eth1:
      addresses:
        - 10.10.20.90/24
    eth2:
      dhcp4: yes

```

Then execute `sudo netplan apply`

# Step 2: Install OpenStack

## Download script to all servers
- Login to root user, Download git & and clone scripts

```sh
$ apt-get -y update && apt-get -y install git-core
$ git clone https://github.com/thaivinhtai/openstack-quick-installer.git /root/openstack
$ cd /root/openstack/scripts
$ chmod -R +x *.sh
```

## Re-config out/vars
Depending on the network topology, edit the values of environment vars in out/vars:

```sh
###################################################################################################
# Network interfaces
## Management Network
export MGNT_INTERFACE=eth0
## Tenant / xvlan Network
export DATA_INTERFACE=eth1
## Provider Network
export EXT_INTERFACE=eth2

## linuxbridge || openvswitch
export NEUTRON_AGENT=linuxbridge

## Assigning IP for CONTROLLER node
export CTL_MGNT_IP=172.25.234.90
export CTL_EXT_IP=192.168.81.165
export CTL_DATA_IP=10.10.20.90

## Assigning IP for COMPUTE1 node
export COM1_MGNT_IP=172.25.234.100
export COM1_EXT_IP=192.168.81.166
export COM1_DATA_IP=10.10.20.100
#
## Assigning IP for COMPUTE2 node
export COM2_MGNT_IP=172.25.234.101
export COM2_EXT_IP=192.168.81.167
export COM2_DATA_IP=10.10.20.101

## Assigning IP for CINDER node
export CIN_MGNT_IP=172.25.234.102
export CIN_DATA_IP=10.10.20.102

## Gateway for EXT network
export GATEWAY_IP_EXT=192.168.81.1
export NETMASK_ADD_EXT=255.255.255.0

## Gateway for MGNT network
export GATEWAY_IP_MGNT=172.25.234.1
export NETMASK_ADD_MGNT=255.255.255.0

## Gateway for DATA network
export GATEWAY_IP_DATA=10.10.20.1
export NETMASK_ADD_DATA=255.255.255.0

## DNS server
export DNS_IP="8.8.8.8"
## NTP server
export NTP_SERVER="europe.pool.ntp.org"

###################################################################################################
# OpenStack settings
## OpenStack API endpoint (public IP or domain)
export PUBLIC_FQDN_CTL=192.168.81.165
export INTER_FDND_CTL=10.10.20.90
# export PUBLIC_FQDN_CTL=192.168.81.90
export MGNT_FQDN_CTL=172.25.234.90
# export MGNT_FQDN_CTL=10.138.0.90
export MGNT_FQDN_COM1=172.25.234.100
export MGNT_FQDN_COM2=172.25.234.101
export MGNT_FQDN_CIN1=172.25.234.102


## Current OpenStack Region
export REGION_NAME="CTU_01"

###################################################################################################
# Credentials variable
export DEFAULT_PASS="admin"

## Admin credentials
export CREDENTIALS_ADMIN_USERNAME="admin"
export CREDENTIALS_ADMIN_PASSWORD="admin"
## Demo project credentials
export CREDENTIALS_DEMO_USERNAME="admin"
export CREDENTIALS_DEMO_PASSWORD="admin"

## Internal services
export RABBIT_PASS="$DEFAULT_PASS"
export MYSQL_PASS="$DEFAULT_PASS"

## OpenStack service credentials
export KEYSTONE_PASS="admin"
export GLANCE_PASS="admin"
export NOVA_PASS="admin"
export NEUTRON_PASS="admin"
export PLACEMENT_PASS="admin"
export METADATA_SECRET="db3c2f34-18c6-11ea-ad25-330a32b12066"
export CINDER_PASS="admin"

## OpenStack database credentials
#Database password of Identity service
export KEYSTONE_DBPASS="admin"
#Database password for Image service
export GLANCE_DBPASS="admin"
#Database password for Compute service
export NOVA_DBPASS="admin"
export NOVA_API_DBPASS="admin"
#Database password for the Networking service
export NEUTRON_DBPASS="admin"
#Database password for the Block Storage service
export CINDER_DBPASS="admin"
```

## Install OpenStack cluster
### Controller

- SSH with `root` user and run scripts

```sh
$ source out/vars
$ ./setup01.sh
$ ./setup02.sh controller
$ ./setup03.sh controller
```

### Compute1 and Compute2

- SSH with `root` user and run scripts

```sh
$ source out/vars
$ ./setup01.sh
$ ./setup02.sh compute{1, 2}
$ ./setup03.sh compute{1, 2}
```

### Block

- SSH with `root` user and run scripts

```sh
$ source out/vars
$ ./setup01.sh
$ ./setup02.sh block1
$ ./setup03.sh block1
```

## Install OpenStack All in One

- SSH with `root` user and run scripts

```sh
$ source out/vars
$ ./setup01.sh
$ ./setup02.sh all-in-one
$ ./setup03.sh all-in-one
```

# Step 3: Test operation
## Create demo VMs
```sh
$ . admin-openrc
# Create Cirros images
$ ./test_glace.sh
# Create base network
$ ./test_neutron.sh

# Modify SECURITY_GROUP_ID match with current project
$ ./test_nova_provider_network.sh
$ ./test_nova_self_network.sh
```

## Login dashboad

- Dashboard: `http://<controller mngt IP>/horizon` or `http://${PUBLIC_FQDN_CTL}/horizon`
- User : `admin / admin`

## Check by command or dashboard

![console](./images/img1.png)
![web](./images/img2.png)

# Credit
Thanks to @PT Studio https://github.com/pt-studio/openstack-queens-labs.git
