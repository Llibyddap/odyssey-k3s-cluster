**Hardware**

*Cluster Computers*

Odyssey X86 Blue J4105
Brand:  Seeed Studio

CPU:    Celeron J4105

RAM:    8 GB

SSD:    128 GB

This project will use three Odyssey computers to build a cluster with a single master node and two worker nodes.

*Manager Computer (docker image)*

`dorowu/ubuntu-desktop-lxde-vnc:focal-lxqt`

All cluster management will be accomplished outside the cluster using the Manager.  This can also be done on Windows 10 but it is substantially more time consuming given the added complexity of the Windwos 10 environment constraints.

**Step 1**

*Operating system*

Download the Ubuntu operating system iso file.  This example uses version 20.04 LTS Server.  We'll flash the iso image onto a USB flash drive using Rufus (flashing was done on a Windows 10 operating system).

**Step 2** (Cluster Computers)

The same USB drive will be used to install the Operating System on all three Odyssey computers.  Note that the only difference will be in setting the server name (`k3s_master01`, `k3s_node01`, `k3s_node2`).

Boot with Ubuntu 20.04 LTS Server USB 3.0 drive..  To do this you'll need to have a keyboard installed, an ethernet connection (for network connection), a monitor to see what you're doing and the USB drive with the operating system.  The USB drive needs to be inserted into the blue USB port.  Once all devices are attached, power on the Odyssey and press the delete key to enter the boot setup menu.

scroll to Save and Exit at the far right side using the right arrow key.  Select the UEFI OS (...USB 3.0...) boot option.  This will automatically reboot the Odyssey using the USB drive with the Ubuntu server image.

Once the system reboots, it will begin the isntallation script which may need to be updated.  Generally, the process involves accepting all defualts unless noted below.

Network Connections
Accept the default connections, but take note of the IP address assigned to the Odyssey.

*Profile*

Here you should enter `k3s_master01` as the server name.  To make things easy, you should select the same user and password when setting up the profile on each of the comptuers (e.g., `USER`).  Note that this user is a sudo user and because we accepted the default configuration we are not using SSH authentication keys - these will be added manually later.  Installing SSH keys from the installation script requires them to be downloaded from either GitHub or LaunchPad.

*SSH Setup*

Check the box to install openSSH server

Upon completion of the installation process, you'll need to manually reboot the system at the bottom of the screen.

**Step 3** (from Manager Computer)

** test ssh...
using another networked computer you can SSH into each of the nodes to confirm they are up and running.

`ssh USER@xxx.xxx.xxx.xxx`

Once loged in to the server you can complete the update process with:

```
sudo apt update
sudo apt upgrade -y
sudo visudo             <-- This will allow you to edit the sudoers file.  
```

Inside the visudo file, scroll to the bottom and add the following line.

`USER  ALL=(ALL) NOPASSWD:ALL`

**Step 4**

Repeat Steps 3 and 4 on the other two Odyssey computers remembering to use the k3s_node01 and k3s_node02 server names.

**Step 5** (from Manager Computer)

from the root directory...  

apt-get install openssh-client git ansible nano -y (add to dependencies)

ssh-keygen (accept defaults including no passphrase)

ssh-copy-id USER@xxx.xxx.xxx.xxx (copy keys over to k3s_master01, k3s_node01 and k3s_node02)

This allows you to ssh into the node without using a password and will be important in automating the cluster deployment with ansible.

**Step 6** (from Manager Computer) 

Install the k3s-ansible play book.

`git clone https://github.com/k3s-io/k3s-ansible.git`

Create a duplicate of the directory called `inventory/sample` with the name `inventory/odyssey`

`cp -R inventory/sample inventory/odyssey`

Edit the `inventory/odyssey/hosts.ini` file with `nano inventory/odyssey/hosts.ini`

```
[master]
xxx.xxx.xxx.xxx       <--  This will be the IP address of k3s_master01

[node]
xxx.xxx.xxx.xxx       <--  This will be the IP address of k3s_worker01
xxx.xxx.xxx.xxx       <--  This will be the IP address of k3s_worker02

[k3s_cluster:children]
master
node
```

`Control-x`, `y` to exit and save your work.

Edit the `inventory/odyssey/group_vars/all.yml` file with `nano inventory/odyssey/group_vars/all.yml`

```
---
k3s_version: v1.17.5+k3s1
ansible_user: USER        <--  This line needs to change from debian to your USER name.
systemd_dir: /etc/systemd/system
master_ip: "{{ hostvars[groups['master'][0]]['ansible_host'] | default(groups['master'][0]) }}"
extra_server_args: ""
extra_agent_args: ""
```

`Control-x`, `y` to exit and save your work.
