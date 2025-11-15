📝 Introduction



This project implements a High Availability (HA) network setup using Linux NIC bonding in active-backup mode. Two physical network interfaces (enp0s3 and enp0s8) are combined into a single logical interface (bond0). If the active NIC fails, network traffic automatically switches to the backup NIC without interruption. This setup ensures continuous connectivity, providing reliability, redundancy, and fault tolerance for critical systems.



🎯 Objectives

* To configure high availability networking using NIC bonding in CentOS 9
* To ensure zero network downtime during NIC failure
* To combine two NICs into one logical bonded interface (bond0)
* To test and verify automatic failover between NICs
* To demonstrate a practical Linux system administration HA setup





🛠️ Tools and Technologies Used

1\. modprobe bonding

Used to load the Linux bonding kernel module, enabling NIC bonding functionality.



2\. nmcli (NetworkManager CLI)

Used to create, configure, and manage network connections, including bond interfaces and slave NICs.



3\. ifenslave 

A utility used for attaching and detaching slave interfaces to bonding devices.



4\. ping

Used to test network connectivity and verify that failover occurs without packet loss.



5\. VirtualBox

Virtualization software used to create CentOS 9 VM with two NICs for testing bonding.



🖥️ System Setup (VirtualBox Configuration)



* CentOS Stream 9 Virtual Machine
* Two network adapters (Host-Only mode recommended)
* Adapter 1 → enp0s3
* Adapter 2 → enp0s8
* Both NICs on the same network (192.168.56.x)



⚙️ Implementation Steps

1\. Check available network interfaces

ip a



2\. Load bonding module

sudo modprobe bonding

lsmod | grep bonding



3\. Make bonding module permanent

echo bonding | sudo tee /etc/modules-load.d/bonding.conf



4\. Create bond0 interface in active-backup mode

sudo nmcli connection add type bond con-name bond0 ifname bond0 mode active-backup



5\. Assign IP to bond0

sudo nmcli connection modify bond0 ipv4.addresses 192.168.56.150/24

sudo nmcli connection modify bond0 ipv4.method manual

sudo nmcli connection up bond0



6\. Add NICs as slaves

sudo nmcli connection add type ethernet con-name bond-slave1 ifname enp0s3 master bond0

sudo nmcli connection add type ethernet con-name bond-slave2 ifname enp0s8 master bond0



sudo nmcli connection modify bond-slave1 ipv4.method disabled

sudo nmcli connection modify bond-slave2 ipv4.method disabled



sudo nmcli connection up bond-slave1

sudo nmcli connection up bond-slave2



7\. Verify bonding configuration

cat /proc/net/bonding/bond0

nmcli device status



🧪 Failover Testing (High Availability Test)

Terminal 1: Start continuous ping

ping 192.168.56.150



Terminal 2: Simulate NIC failure

sudo nmcli device disconnect enp0s8



Expected Output



Ping continues without interruption



Bond fails over automatically to the backup NIC (enp0s3)



No packet loss → High Availability confirmed



Restore NIC

sudo nmcli device connect enp0s8

