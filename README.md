easyOVS
=======

Provide smarter and powerful operation on OpenvSwitch bridges in OpenStack.

version 0.4

#What is easyOVS
easyOVS provides a more convenient and fluent way to operate your 
[OpenvSwitch](http://openvswitch.org) bridges in OpenStack platform,
such as list their ports, dump their flows and add/del some flows in a smart 
style with color!

If using in OpenStack environment (Currently tested from the Havana to Juno 
release), easyOVS will associate the ports with the vm MAC/IP and VLAN Tag information, and the iptables rules for vm.

#Installation and Usage
Download the latest version and install.

`git clone https://github.com/yeasy/easyOVS.git && sudo bash ./easyOVS/util/install.sh`

After the installation, start easyovs with

`sudo easyovs`

easyOVS will show an interactive CLI, which supports command suggestions and formatted colorful output.

##Enable OpenStack Feature
To integrate the port information collected from OpenStack, 
please set the authentication information in your environment:
e.g., 
```sh
export OS_USERNAME=demo
export OS_TENANT_NAME=demo
export OS_PASSWORD=demo
export OS_AUTH_URL=http://127.0.0.1:5000/v2.0/
```
Otherwise, set the information into etc/easyovs.conf files.
```sh
[OS]
auth_url = http://127.0.0.1:5000/v2.0
username = demo
password = admin
tenant_name = demo
```sh


##Upgrade or Delete
If you wanna upgrade easyOVS from a previous version, just run

`sudo bash ./easyOVS/util/install.sh -u`

If you wanna to remove the package from the system

`sudo bash ./easyOVS/util/install.sh -d`

#Documentation

##CLI Commands

###help
Show the available commands and some usage examples.

###list
List the available bridges. The output would look like
```
 EasyOVS> list
s1
 Port:		s1-eth2 s1 s1-eth1
 Interface:	s1-eth2 s1 s1-eth1
 Controller:ptcp:6634 tcp:127.0.0.1:6633
 Fail_Mode:	secure
s2
 Port:		s2 s2-eth3 s2-eth2 s2-eth1
 Interface:	s2 s2-eth3 s2-eth2 s2-eth1
 Controller:tcp:127.0.0.1:6633 ptcp:6635
 Fail_Mode:	secure
s3
 Port:		s3-eth1 s3-eth3 s3-eth2 s3
 Interface:	s3-eth1 s3-eth3 s3-eth2 s3
 Controller:ptcp:6636 tcp:127.0.0.1:6633
 Fail_Mode:	secure
```

###show
`EasyOVS> [bridge|default] show`

Show the ports information of a given bridge. The output would look like
```
 EasyOVS> br-int show
br-int
Intf                Port        Vlan    Type        vmIP            vmMAC
int-br-eth0         15
qvo260209fa-72      11          1                   192.168.0.4     fa:16:3e:0f:17:04       
qvo583c7038-d3      2           1                   192.168.0.2     fa:16:3e:9c:dc:3a       
qvo8bf9cba2-3f      9           1                   192.168.0.5     fa:16:3e:a2:2f:0e
qvod4de9fe0-6d      8           2                   10.0.0.2        fa:16:3e:38:2b:2e       
br-int              LOCAL               internal
```

###dump
`EasyOVS> [bridge|default] dump`

Dump flows in a bridge. The output would look like

```
EasyOVS> s1 dump
ID TAB PKT       PRI   MATCH                                                       ACT
0  0   0         2400  dl_dst=ff:ff:ff:ff:ff:ff                                    CONTROLLER:65535
1  0   0         2400  arp                                                         CONTROLLER:65535
2  0   0         2400  dl_type=0x88cc                                              CONTROLLER:65535
3  0   0         2400  ip,nw_proto=2                                               CONTROLLER:65535
4  0   0         801   ip                                                          CONTROLLER:65535
5  0   2         800
```

###addflow
`EasyOVS> [bridge|default] addflow [match] actions=[action]`

Add a flow into the bridge, e.g.,

`EasyOVS> br-int addflow priority=3 ip actions=OUTPUT:1`

###delflow
`EasyOVS> [bridge|default] delflow id1 id2...`

Delete flows with given ids (see the first column of the `dump` output).


###set
`EasyOVS> bridge set`

Set the default bridge. Then you will go into a bridge mode, and can ignore the bridge parameter when using the
command.
```
EasyOVS> set br-int
Set the default bridge to br-int.
EasyOVS: br-int> 
```

###exit
`EasyOVS> exit`

Exit from the bridge mode, or quit EasyOVS if already at the top level.

###get
`EasyOVS> get`

Get the current default bridge.
```
EasyOVS: br-int> get
Current default bridge is br-int
```

###ipt
`EasyOVS> ipt vm_ip1, vm_ip2...`

Show the related iptables rules of the given vms.
```
EasyOVS> ipt 192.168.0.2 192.168.0.4
## IP = 192.168.0.2, port = qvo583c7038-d ##
    PKTS	SOURCE          DESTINATION     PROT  OTHER               
#IN:
     672	all             all             all   state RELATED,ESTABLISHED
       0	all             all             tcp   tcp dpt:22          
       0	all             all             icmp                      
       0	192.168.0.4     all             all                       
       3	192.168.0.5     all             all                       
       8	10.0.0.2        all             all                       
   85784	192.168.0.3     all             udp   udp spt:67 dpt:68   
#OUT:
    196K	all             all             udp   udp spt:68 dpt:67   
   86155	all             all             all   state RELATED,ESTABLISHED
    1241	all             all             all                       
#SRC_FILTER:
   59163	192.168.0.2     all             all   MAC FA:16:3E:9C:DC:3A
## IP = 192.168.0.4, port = qvo260209fa-7 ##
    PKTS	SOURCE          DESTINATION     PROT  OTHER               
#IN:
      73	all             all             all   state RELATED,ESTABLISHED
       0	all             all             tcp   tcp dpt:22          
       0	all             all             icmp                      
       0	192.168.0.2     all             all                       
       0	192.168.0.5     all             all                       
       0	10.0.0.2        all             all                       
   11331	192.168.0.3     all             udp   udp spt:67 dpt:68   
#OUT:
   30034	all             all             udp   udp spt:68 dpt:67   
   11377	all             all             all   state RELATED,ESTABLISHED
      12	all             all             all                       
#SRC_FILTER:
    9859	192.168.0.4     all             all   MAC FA:16:3E:0F:17:04

```

###sh
`EasyOVS> sh cmd`

Run the system cmd locally, e.g., using ls -l to show local directory's content.
```
EasyOVS> sh ls -l
total 48
drwxr-xr-x. 2 root root 4096 Apr  1 14:34 bin
drwxr-xr-x. 5 root root 4096 Apr  1 14:56 build
drwxr-xr-x. 2 root root 4096 Apr  1 14:56 dist
drwxr-xr-x. 2 root root 4096 Apr  1 14:09 doc
drwxr-xr-x. 4 root root 4096 Apr  1 14:56 easyovs
-rw-r--r--. 1 root root  660 Apr  1 14:56 easyovs.1
drwxr-xr-x. 2 root root 4096 Apr  1 14:56 easyovs.egg-info
-rw-r--r--. 1 root root 2214 Apr  1 14:53 INSTALL
-rw-r--r--. 1 root root 1194 Apr  1 14:53 Makefile
-rw-r--r--. 1 root root 3836 Apr  1 14:53 README.md
-rw-r--r--. 1 root root 1177 Apr  1 14:53 setup.py
drwxr-xr-x. 2 root root 4096 Apr  1 14:09 util
```

###quit
Input `^d` or `quit` to exit EasyOVS.

##Options
###-h
Show the help message on supported options, such as
```
$ easyovs -h
Usage: easyovs [options]
(type easyovs -h for details)

The easyovs utility creates operation CLI from the command line. It can run
given commands, invoke the EasyOVS CLI, and run tests.

Options:
  -h, --help            show this help message and exit
  -c, --clean           clean and exit
  -m CMD, --cmd=CMD     Run customized commands for tests.
  -v VERBOSITY, --verbosity=VERBOSITY
                        info|warning|critical|error|debug|output
  --version
```

###-c
Clean the env.

###-m
Run the given command in easyovs, show the output, and exit.

E.g. `easyovs -m 'br-int dump'`.

###-v
Set verbosity level.

###--version
Show the version information.

#Features
* Support OpenvSwitch version 1.4.6 ~ 1.11.0.
* Support most popular Linux distributions, e.g., Ubuntu,Debian, CentOS and Fedora.
* Format the output to make it clear and easy to compare.
* Show the OpenStack information with the bridge port (In OpenStack environment).
* Delete a flow with its id.
* Show iptables in formated way of given vm IPs.
* Smart command completion, try tab everywhere.
* Support colorful output.
* Support run local system commands.
* Support run individual command with `-m 'cmd'`

#Credits
Thanks to the [OpenvSwitch](http://openvswitch.org) Team, [Mininet](http://mininet.org) Team and [OpenStack](http://openstack.org) Team, who gives helpful implementation example and useful tools.