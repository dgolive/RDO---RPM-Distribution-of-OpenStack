# OpenStack PackStack
Details to Deploy RDO

OPENSTACK - PACKSTACK

yum install net-tools lspci nc mlocate wget vim tcpdump telnet git traceroute ntp yum-utils -y

Create a VG cinder-volumes 

# RDO Installation
PREPARATION

vim /etc/environment 
LANG=en_US.utf-8
LC_ALL=en_US.utf-8

sudo systemctl disable firewalld
sudo systemctl stop firewalld
sudo systemctl disable NetworkManager
sudo systemctl stop NetworkManager
sudo systemctl enable network
sudo systemctl start network

Change SELINUX from permissive to permissive (just dev env)

sudo yum update
sudo yum install -y centos-release-openstack-stein
sudo yum update -y
sudo yum install -y openstack-packstack

reboot

#check interfaces
ip l | grep '^\S' | cut -d: -f2

OVS NIC (ie: br-ex)
#cat /etc/sysconfig/network-scripts/ifcfg-br-ex
TYPE=OVSBridge
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
NAME=em1
UUID=cd071c15-e38b-49cc-980d-207c0c4ddaa2
DEVICE=em1
DEVICETYPE=ovs
ONBOOT=yes
DNS1=10.11.12.246
IPADDR=10.10.5.11
PREFIX=22
GATEWAY=10.10.5.254
OVSBOOTPROTO=dhcp
OVSDHCPINTERFACES=em2

#cat /etc/sysconfig/network-scripts/ifcfg-em2

TYPE=OVSPort
BOOTPROTO=none
NAME=em2
UUID=804f3bfc-2d46-4a4b-b225-146950672da4
DEVICE=em2
DEVICETYPE=ovs
OVS_BRIDGE=em1
ONBOOT=yes
###


# DEPLOY RDO with External Network
packstack --allinone --provision-demo=n --os-neutron-ovs-bridge-mappings=extnet:br-ex --os-neutron-ovs-bridge-interfaces=br-ex:em1 --os-neutron-ml2-type-drivers=vxlan,flat

#BASIC INSTALL using config file
packstack --allinone --provision-demo=n ou sudo packstack --answer-file=FILE


RESULTADO:
**** Installation completed successfully ******

Additional information:
 * A new answerfile was created in: /root/packstack-answers-20160606-135853.txt
 * Time synchronization installation was skipped. Please note that unsynchronized time on server instances might be problem for some OpenStack components.
 * File /root/keystonerc_admin has been created on OpenStack client host 10.16.203.100. To use the command line tools you need to source the file.
 * To access the OpenStack Dashboard browse to http://10.16.203.100/dashboard .
Please, find your login credentials stored in the keystonerc_admin in your home directory.
 * To use Nagios, browse to http://10.16.203.100/nagios username: nagiosadmin, password: 1ed10ce2490d4f2d
 * The installation log file is available at: /var/tmp/packstack/20160606-135852-aYugBZ/openstack-setup.log
 * The generated manifests are available at: /var/tmp/packstack/20160606-135852-aYugBZ/manifests


# Config env
neutron net-create external_network --provider:network_type flat --provider:physical_network extnet  --router:external

neutron subnet-create --name public_subnet --enable_dhcp=False --allocation-pool=start=10.10.128.50,end=10.10.128.250 --gateway=10.10.159.254 external_network 10.10.128.0/19

neutron router-create router1
neutron router-gateway-set router1 external_network

ovs-vsctl show

# IMPORTAR IMAGEM
export CBD_LATEST_IMAGE=cloudbreak-deployer-163-2017-02-22.img 
export OS_IMAGE_NAME= cloudbreak-deployer

glance image-create --name "$OS_IMAGE_NAME" --file "$CBD_LATEST_IMAGE" --disk-format qcow2 --container-format bare --progress


ANSWER FILE
run packstack --answer-file=./generated-answer.txt

Verificar versão

# cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)


[root@aqt2 instances]# virsh list
 Id Name State
----------------------------------------------------
 4 instance-00000017 running


[root@aqt2 instances]# virsh domblklist 4
Target Source
------------------------------------------------
vda /dev/disk/by-path/ip-10.10.28.1:3260-iscsi-iqn.2010-10.org.openstack:volume-48d79ed7-fa16-4436-a349-613f0781e1e9-lun-0

 nova aggregate-create zona1_home zona1_home
 nova aggregate-add-host 1 aqt2


[root@aqt2 instances(keystone_admin)]# nova host-list
+-----------+-------------+----------+
| host_name | service | zone |
+-----------+-------------+----------+
| aqt2 | cert | internal |
| aqt2 | consoleauth | internal |
| aqt2 | scheduler | internal |
| aqt2 | conductor | internal |
| aqt2 | compute | nova |
+-----------+-------------+----------+
[root@aqt2 instances(keystone_admin)]# cinder availability-zone-list
+------+-----------+
| Name | Status |
+------+-----------+
| nova | available |
+------------------+


references:
https://www.rdoproject.org/install/packstack/
https://www.rdoproject.org/networking/neutron-with-existing-external-network/
http://boxofclue.com/presentations/centos_packaging/#/
https://docs.openstack.org/project-deploy-guide/openstack-ansible/newton/targethosts-prepare.html
