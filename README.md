# Linux_cheatsheet

# Networking,Services and system updates

## Network files and commands


- ### Interface configuration files
  - **/etc/nsswitch.conf**  - nsswitch. conf file is used to configure which services are to be used to determine information such as hostnames, password files, and group files
  
  - **/etc/hostname**   - Where you define system ip-address and system hostname
  
  - **/etc/sysconfig/network**  -  /etc/sysconfig/network file specifies additional information that is valid to all network interfaces on the system.
  
  - **/etc/sysconfig/network-scripts/ifcfg-nic** - which controls the first Ethernet network interface card or NIC in the system. In a system with multiple NICs, there are multiple ifcfg-ethX files
  
  - **/etc/resolv.conf** - DNS server

 
- ### Network Commands
  - ping 
  - ifconfig
  - ifup / ifdown
  - netstat
  - tcpdump



## NIC information

- You can find out your nic information using **ifconfig** 

  ```bash
  [root@localhost ~]# ifconfig
  enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.135.213  netmask 255.255.255.0  broadcast 192.168.135.255
          inet6 fe80::df40:d2d6:b4f2:d9d4  prefixlen 64  scopeid 0x20<link>
          ether 08:00:27:f3:2c:87  txqueuelen 1000  (Ethernet)
          RX packets 395  bytes 237283 (231.7 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 294  bytes 27937 (27.2 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

  lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
          inet 127.0.0.1  netmask 255.0.0.0
          inet6 ::1  prefixlen 128  scopeid 0x10<host>
          loop  txqueuelen 1000  (Local Loopback)
          RX packets 128  bytes 11136 (10.8 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 128  bytes 11136 (10.8 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

  virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
          inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
          ether 52:54:00:5a:a2:eb  txqueuelen 1000  (Ethernet)
          RX packets 0  bytes 0 (0.0 B)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 0  bytes 0 (0.0 B)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

  [root@localhost ~]#
  ```
- Find information about the **NIC**
  ```bash
  ethtool [interface name]
  ```
  
- Other NICs
  - lo = The loopback device is a special, virtual network interface that your computer uses to communicate with itself. It is used mainly for diagnostics and troubleshooting, and to connect to servers running on the local machine.
  
  - The virbr0 , or "Virtual Bridge 0" interface is used for NAT (Network Address Translation). It is provided by the libvirt library, and virtual environments sometimes use it to connect to the outside network. It was likely bundles with a VM software you installed at some point.

## NIC Bonding


### what is NIC Bonding?
  - NIC bonding is also known as Network bonding. It can be defined as the aggregation or combination of multiple NIC into a single bond interface.
  - It's main purpose is to provide high availability and redundancy.


### How it be done

1. Create Bond Interface File
  - vi /etc/sysconfig/network-scripts/ifcfg-bond0
  - next add following parameters
    ```bash
    DEVICE=bond0
    TYPE=Bond
    NAME=bond0
    BONDING_MASTER=yse
    BOOTPROTO=yes
    IPADDR=[$ip-addr]
    NETMASK=[subnet-mask]
    GATEWAY=[gateway_$ip-addr]
    BONDING_OPTS="mode=5 miimon=100"
    ```
2. Save and exite the file

  - The bonding options details are can be found on the following table
    ![PortBondingOptions](https://imgur.com/a/RpgWFGQ)

  - **miimon** 
    Specifies the MII link monitoring frequency in milliseconds. This determine how often the link state of each slave is inspected for link failures.

3. Edit the other NIC FILES(enp0s3 and enp0s8)
  - vi /etc/sysconfig/network-scripts/ifcfg-enp0s8
  - Next add the following parameters.
    ```bash
    TYPE=Ethernet
    BOOTPROTO=none
    DEVICE=[Device_name]
    ONBOOT=yes
    HWADDR="**MAC_ADDR**"
    MASTER=bond0
    SLAVE=yes
    ```
4. Save and exit the file.
5. Do this for other NICs as well.
  
6. Restart the Network Service. 
  ```bash
      systemctl restart network  
  ```
 
7. Test and verify the configeration
  ```bash
      ifconfig
  ```
9. Use this command to view bond interface settings like bonding mode & slave interface
  ```bash
      cat /prov/net/bonding/bond0
  ```
10. We have done it. :)


## Network Utilities




## FTP - File Transfer Protocol

- The File Transfer Protocol is standard network protocol used for the transfer of computer files between a client and server between a client and server on a computer network. FTP is built on a client-server model architecture using seperate control and data connections between the client and the server.


### How to setup a FTP server
1. First you need to install the package.
```bash
yum install vsftpd
```
2. next edit the configuration file.
```bash
vi /etc/vsftpd/vsftpd.conf
```
  - change following things
    - anonymous_enable=`NO`
    - uncommented these things
      1. ascii_upload_enable=YES
      2. ascii_download_enable=YES
      3. ftpd_banner=enter your welcome massage
3. add
  - use_localtime=YES

4. start ftp service
```bash
systemctl start vsftpd
```
5. enable ftp service
```bash
systemctl enable vsftpd
```
6. add firewall rule go through [this](https://linuxconfig.org/redhat-8-open-ftp-port-21-with-firewalld)




## SCP - Secure Copy Protocol

- The Secure Copy Protocol oi "SCP" helps to transfer computer files securely from a local to a remote host. It is somewhat similar to the File Transfer Protocol "FTP", but it adds security and authentication.

- Transfer file to the remote server through SCP
```bash
scp [file_name] remote_username@$ip:/remote/directory
```
- To copy a directory from a local to remote system, use the -r option:
```bash
scp -r /local/directory remote_username@$ip:/remote/directory
```

## rsync - Remote Synchronization

- rsync is a utility for efficiently transferring and synchronizing files within the same computer or to a remote computer by comparing the modification time and size of files
- rsync is a lot of faster than ftp or scp
-Th9s utility is mostly used to backup the file and directories from one server to atnother 
- Default rsync Port = 22 (sane as SSH)

### usage

- basic syntax
  ```bash
  rsync option source destination
  ```

- rsync a file on a local machine
  ```bash
  rsync -zvh backup.tar /location/
  ```
- rsync a directory on a local machine
  ```bash
  rsync -azvh /directory/ /backup_dir/
  ```
- rsync a file to a remote machine
  ```bash
  rsync -avz backup.tar iafzal@$ip:/tmp/backups
  ```
- rsync a file from a remote machine
  ```bash
  rsync -avzh username@$ip:/file_location/ destination/
  ```




## Create a local reposatary yum server

1. Copy the packages to local reposatary.
2. After coping the packages. remove **/etc/yum.repos.d**.
3. Create a new file **local.repo.d** and insert following things.
  ```bash
  [centos7]
  name=centos7
  baseurl=file:///filename_which_copy_the_packages
  enabled=1
  gpgcheck=0
  ```
4. Create the reposatary
  ```bash
    createrepo /filename_which_copy_the_packages/
   ```
5. Clean all old cache.
  ```bash
    yum clean all
  ```
6. Confirm your job.
  ```bash
    yum repolist all
  ```

## Advanced Package Management

- Install a package when you are online
  ```bash
  yum install [package_name]
  ```
- remove a package when you are online
  ```bash
  yum remove [package_name]
  ```

- install a package when you have a .rpm file
  ```bash
  rpm -hiv package_name.rpm
  ```
- get information about specific package
  ```bash
  [root@localhost pkg]# rpm -qi nmap
  Name        : nmap
  Epoch       : 2
  Version     : 7.92
  Release     : 1
  Architecture: x86_64
  Install Date: Tue 30 Aug 2022 11:57:52 PM +0530
  Group       : Applications/System
  Size        : 28008919
  License     : https://nmap.org/man/man-legal.html
  Signature   : (none)
  Source RPM  : nmap-7.92-1.src.rpm
  Build Date  : Sat 07 Aug 2021 09:10:33 PM +0530
  Build Host  : localhost
  Relocations : (not relocatable)
  URL         : https://nmap.org
  Summary     : Network exploration tool and security scanner
  Description :

  Nmap ("Network Mapper") is a free and open source utility
  for network exploration or security auditing. Many systems and network
    administrators also find it useful for tasks such as network
  inventory, managing service upgrade schedules, and monitoring host or
  service uptime. Nmap uses raw IP packets in novel ways to determine
  what hosts are available on the network, what services (application
  name and version) those hosts are offering, what operating systems
  (and OS versions) they are running, what type of packet
  filters/firewalls are in use, and dozens of other characteristics. It
  was designed to rapidly scan large networks, but works fine against
  single hosts. Nmap runs on all major computer operating systems, and
  both console and graphical versions are available.
  [root@localhost pkg]#
  ```
- remove package using **rpm**
  ```bash
    rpm -e [package_name]
  ```

- Find congigeration files of the package
  ```bash
    rpm -qc [package_name]
  ```

- Find a package of specific command or tool.
  ```bash
  rpm -qf /full/path/to_command
  ```
  - u can search the full path using **which** command ;)

## SSH - Secure shell

- administrators can log into remote devices, execute commands, move files between devices, and more through this commands.

1. First we need to install the package
  ```bash
    yum install sshd
  ```
2. Next open the configeration file and edit it
  ```bash
    vi /etc/ssh/sshd_conf
  ```
3. 

## DNS - Domain Name System

### Setup the DNS server

1. Install the package
  ```bash
    yum install bund bind-utils -y
  ```
2. Edit the configuration file
  ```bash
    vi /etv/named.conf
  ```
```bash
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrators Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
        listen-on port 53 { 127.0.0.1; $serverip; }; #Add Server ip to here
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { localhost; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.root.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "add_here.domain" IN {       #Add your domain 
        type master;        #edit
        file "forward.domain";  #Edit 
        allow-update { none; }; #Add this line
};

zone "1.168.192.in-addr.arpa" IN {  # Add 
type master;                        # Revers
file "reverse.domain";              # settings
allow-update { none; };
};

include "/etc/named.rfc1912.zones";     
include "/etc/named.root.key";          
```
3. Next create the zone files.
  
  - Go to the var/named and crate the zone files.
  ```bash
    cd var/etc
    touch forward.domain
    touch revers.domain
  ```
4. Modify the zone files.
  - vi the forwad zone file and add following files.
  ```bash
   $TTL 86400
   @ IN SOA masterdns.***add_here.domain***. root.***add_here.domain***. ( #edited
        2011071001 ;Serial
        3600 ;Refresh
        1800 ;Retry
        604800 ;Expire
        86400 ;Minimum TTL
   )
   @    IN NS       ***masterdns.add_here.domain***. # edited
   @    IN A        ***$your_server_ipAddress***    # edited
   masterdns      IN A ***$your_server_ipAddress***  #  edited
   clienta        IN A 192.168.1.231     # add your
   clientb        IN A 192.168.1.230     # clients
  ``` 
  - vi the forwad zone file and add following files.
  ```bash
     $TTL 86400
     @ IN SOA masterdns.***add_here.domain***. root.***add_here.domain***. (   #edited
          2011071001 ;Serial
          3600 ;Refresh
          1800 ;Retry
          604800 ;Expire
          86400 ;Minimum TTL
     )
     @    IN NS masterdns.***add_here.domain***. # edited
     @    IN PTR ***add_here.domain***.
     masterdns     IN A ***$your_server_ipAddress*** # edited
     29     IN PTR masterdns.add_here.domain.
     240    IN PTR clienta.***add_here.domain***.  # add your
     241    IN PTR clientb.***add_here.domain***.  # clients
  ```
5. Start the DNS server
  ```bash
    systemctl start named
    systemctl enable named
  ```
6. Configuring Permissions, Ownership, and SELinux
  ```bash
    chgrp named -R /var/named
    chown -v root:named /etc/named.conf
    restorecon -rv /var/named
    restorecon /etc/named.conf
  ```
7. Test DNS configeration and zone files
  ```bash
    named-checkconf /etc/named.conf
    named-checkzone lab.local /var/named/forward.lab
    named-checkzone lab.local /var/named/reverse.lab
  ```
8. Add DNS Server information to network file 
  ```bash
    vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
    DNS=$Server_ipAddress
  ```
9. Modify /etc/resolv.conf
  ```bash
    namesever $Server_ipAddress
  ```

10. Test DNS server using these commands.
  ```bash
  dig 
  nslookup
  ```

# Network Time Protocol (ntp)

- Configuration file = /etc/chronyd.conf
- Log file = /var/log/chrony
- program command = chronyc.

1. Install the package.
  ```bash
    yum install chrony
  ```
2. Configure the package
    - Go to the configeration file = vi /etc/chrony.conf

3. Add your server
  ```bash
  server 8.8.8.8   # enter your server_name
  ```
4. Start the service
  ```bash
  systemctl start chronyd
  ```
5. Run the programme with ***chronyc*** and type ***help*** for more. 

- Everytime you edite the configeration file you have to restart the service.

# System Utility Command - timedatectl

- List of all thime zones
  ```bash
    timedatectl list-timezones
  ```
- Set time zone
  ```bash
    timedatectl set-timezone [timezone]
  ```
- To set date
  ```bash
    timedatectl set-time YYY-MM-DD
  ```
- To set time
  ```bash
    timedatectl set-time 'YYYY-MM-DD HH:MM:SS'
  ```
- To start automatic time synchronization with a remote NTP server
  ```bash
    timedatectl set-ntp true
  ```

# MailServer - Sendmail
- This server use for send and receive emails.
- Files
  - /etc/mail/sendmail.mc
  - /etc/mail/sendmail.cf

- Setup a mail server
  1. Install the package
  ```bash
    yum install sendmail
    yum install sendmail-cf # Configeration of sendmail package
  ```
  - The all configerations are located in '/etc/mail/'
  2. edit the ***/etc/mail/sendmail.mc*** file
    - Add your host name
  
  3. Start the service 
    ```bash
      systemctl start sendmail
    ```
  4. Send mail using mailserver
    ```bash
      mail -s [subject] receivers_mail_address
    ```

# WEB Server (Apache-HTTPs) 



# Centaral loger (rsyslog)

- This server is use for generate or collect logs from other servers
- package name is **rsyslog**
- configeration file =  /etc/rsyslog.conf
- Service
  ```bash
    systemctl restart rsyslog
  ```

- Setup the Server
1. First you need to install package
  ```bash
    yum install rsyslog
  ```
2. Edit the configeration file
  ```bash
    vi /etc/rsyslog.conf
  ```
3. Start the Service
  ```bash
    systemctl start rsyslog
  ```

# Tracing Network Traffic (tracerout)

- Traceroute is the route tracing tool used on Unix-like Operating Systems.
```bash
  tracerout [www.google.com]
```
# 

# Cockpit
- Cockpit is a web based linux administration tool.

- Installation
1. yum install cockpit -y

2. Start the service
  ```bash
    systemctl start/enable cockpit
  ```
- Next connect your server through cockpit

  - Open your browser and serch
  
      **https://[$your_server_ip]:9090**
  
  - Now you can login to your user.
