https://www.reddit.com/r/linuxadmin/comments/4n70ku/advice_for_starting_a_job_in_this_field/d42plhv/

The short version is: expecting that you'll have the experience to call yourself a "senior" admin within the span of a few months isn't realistic. There are simply a whole lot of topics that take a long time to cover, and if I list them here, you'll stop reading (so I list them later ;)

Some of his choices aren't great, either, like suggesting that people set up JBoss Wiki so that they set up tomcat. If you're going to set up tomcat, do something useful with it, like Jenkins. And that, I think, is the same kind of inexperience that leads someone to believe that his list will make a senior system admin.

So... I have a slightly different list of projects that I'd suggest someone undertake in order to build a network that's actually useful in the real world. That list is followed by a list of basic topics that should be reviewed, and should serve as a starting point for continued study.

You'll need a computer with support for virtualization, preferably with multiple hard drives for a RAID (or zfs, or btrfs) storage array, and with a UPS.

##1.	Your first server
1.	KVM - Install a minimal operating system with support for virtualization.

2.	Power management with NUT - Install the NUT server and configure monitoring of the UPS. Unplug the UPS and verify that the system shuts down safely when its battery reaches the charge level you defined, and that the system comes back online when you plug the UPS back in.

##2.	Your first VMs
1.  Documenting your work with a Wiki - Create a new VM and install a minimal OS. Here, install the simplest wiki you can find. I like MoinMoin. For all of the remaining exercises, document everything that you do. Document the location of useful how-tos and other documentation that you find. Document the steps you take setting up a service. Document the details you'd like to come back and revisit later. Records are your friends.
    
2.	Managing changes and setup with Ansible - This one doesn't require a server, per se. You can install ansible on your    own computer or a new VM, according to your preference. Wherever you install it, use mercurial or git to keep track of the changes you make to the ansible repository. Your goal with ansible is to record all of the changes you make to any further systems in ansible, so that repetition of your work is automated. Each change that you make to a system should be recorded in an ansible playbook. When you are comfortable with ansible, you should be able to set up a server once, then destroy it, and rebuild it using only your ansible playbook. When you reach that level of comfort, you should never be making changes directly to your servers; instead, make the changes in ansible and push the changes out from there. Don't worry if you don't get there immediately, but this is ultimately your goal.

##3.	Network basics
1.	ISC DHCP - Set up a new VM and configure a basic instance of dhcpd. (Your KVM server should have a static network configuration. In production, VMs may not be ideal for hosing dhcpd.)
    
2.	Spacewalk - Set up a new VM and install spacewalk. Configure the server to host packages for your systems locally, to speed up the installation of additional systems.

##4.	User management
1.	DNS - Set up another new VM and install named. Configure your systems to use this instance as their name server. Set up a new domain so that you can name your local systems.
    
2.	LDAP - Set up another VM and install 389-ds. Use the server's "console" application to create new users and assign passwords. Set up one or more systems to authenticate users from LDAP.
    
3.	Kerberos - Add krb5-server to the LDAP vm. Create principals for each of the users that you've set up in LDAP, and add passwords to them. Modify any systems that are authenticating using LDAP so that they authenticate using kerberos. Add host keytabs to those servers. You should be able to log in to one, and from there, log in to another kerberos-authenticated system using your kerberos ticket instead of a password.
    
4.	Certificate Authority - Use openssl to create a new certificate authority, and then create and sign some certificates. OpenVPN's easy-rsa is a convenient wrapper for this task.
    
5.	FreeIPA - Set up a new VM and install FreeIPA. This will replace the DNS, LDAP, Kerberos, and certificate servers you've setup, so don't refer to those in this VM. Set up your users in the FreeIPA interface. Set up any servers that refer to the individual DNS, LDAP, and Kerberos servers to use the FreeIPA server instead. Compare the operation of FreeIPA to operation of the individual servers and then delete the VMs that host the individual DNS, LDAP, Kerberos, and certificate servers.

##5.	Redundant Shared storage
1.	SQL - Set up a new VM with PostgreSQL. Set up a second new VM with PostgreSQL, and use pgpool-II to replicate data between them. Reconfigure spacewalk to use PostgreSQL.
    
2.	File server - Set up a new VM and set up GlusterFS. Create a new volume. Mount this directory on your other VMs as /home.

##6.	Backups 

Set up another new VM and install rsnapshot. Set up backups for all of your VMs. Schedule daily backups using cron. Dump your PostgreSQL databases to plain-text files before backing them up.

##7.	Collaboration
1.	Email - Set up another new VM and install Courier. Configure SMTP and IMAP. Configure Thunderbird on your workstation to send and receive mail through the Courier system.
    
2.	Calendar - Set up another new VM and install SOGo. Set up the Thunderbird extensions. Delete your Thunderbird profile, and set up your account from scratch. The client should now have email as well as a calendar and address book that can be shared with other users.

##8.	Monitoring
    
Set up another new VM and install Nagios. Monitor each system to make sure it's up. Monitor each network service to make sure that actually responds appropriately to requests. Create service accounts so that the monitoring system can authenticate to services that require authentication.

##9.	Centralized logs 
        
Set up another new VM and install logstash. Configure each of your VMs to send a copy of all log messages to the log server.

##10.	Automation with Jenkins 

Set up another new VM and install Jenkins. Use Jenkins to schedule tasks. For instance, take your backup jobs out of cron and use Jenkins to run them, and to report success or failure on its dashboard. Turn off a non-critical VM and run backups to observe a failure report.

11.	Building packages in mock 

Set up another new VM. Install httpd, mock and GnuPG. Grab a package from Fedora that's not available in CentOS. Use mock to rebuild the package, and then sign the package using rpm and gpg. Create a directory in the web server root for packages and copy the signed packages there. Use 'createrepo' to set up the metadata used by yum. Configure a one of your systems to use that HTTP server location as a package source and verify that the package can be installed.

12.	Track issues with RT4 

Set up another new VM and install Best Practical's RT ticket tracking system.

# Core concepts are Storage, Security, and Networking.

##1.	Block Storage
1.	RAID 0, 1, 5, 6, 10 - You should understand the basics of how each array type work, how their performance changes in relation to the number of disks in the array, and how each type respond to the loss of one or more disks. Different deployment types will call for different storage arrays.
    
2.	LVM - Learn to reserve chunks of storage for filesystems and other higher level storage use. LVM has a variety of uses, including resizing filesystems, snapshots for backups, and providing low-overhead storage for virtualization.
    
3.	Hierarchical storage array - Learn to combine a fast storage device, such as an SSD or SSD RAID, with a slower array, such as HDD RAID. This can provide a good balance of speed for commonly accessed data with low cost for bulk storage.
    
4.	Hardware RAID vs Software RAID - Hardware arrays should provide a battery-backed write cache so that in the event of power loss, a write that spans disks can be completed when power is restored. Software arrays may be faster, more flexible, more portable between systems, more consistent, and generally easier to manage. However, because they lack the battery backed write cache, it's essential that you provide backup power, monitor it, and shut down safely if power is lost.
    
5.	btrfs and ZFS - These filesystems combine the features of RAID, LVM, and traditional filesystems. The increased integration means that they can rebuild faster in the event of a disk loss, and management is typically simpler than the traditional stack. Additionally, they provide checksums for data and metadata, so they can detect and correct corruption in situations where RAID arrays can't. 

##2.	File Storage
1.	ext4 - The default Linux filesystem in most distributions. Like most modern filesystems, it supports extended attributes, ACLs for complex permission sets, and a journal for integrity.
    
2.	XFS - This filesystem is the default for Red Hat Enterprise Linux 7, and offers better performance in some workloads.
    
3.	SMB - A network filesystem appropriate for individual users to connect to a server. Connections are authenticated initially, and activity on that connection continues to use the authorization of the user which initially provided credentials.
    
4.	NFS - A network filesystem appropriate for trusted multi-user systems. Early versions authenticated the client host instead of the client user, and trusted that host to send correct information about which user initiated each request to the filesystem. Trust allowed multiple users to share the same connection to a server. Later, kerberos support was added to the filesystem so that the server could authenticate each request, decreasing its reliance on simple trust mechanisms.
    
5.	iSCSI - Can be used to access a block storage device, such as a disk or RAID array, over a network. Clients can use the block device as backing for a VM, or a traditional filesystem. Multiple clients can use the block device with clustered filesystems like GFS2.
    
6.	DRBD - Network-replicated block device. Can be used for active/passive failover systems or for for clustered filesystems.
    
7.	GlusterFS - Clustered filesystem where individual files are replicated and/or distributed among servers, rather than raw blocks. 

##3.	Higher Level Data Storage
1.	Mercurial and Git - Use these tools to track changes in files, revert to earlier versions, inspect the history of changes, and to copy data and changes from place to place.
    
2.	PostgreSQL - SQL allows users to define structures in which to store data. Unlike files, where the server simply accepts and stores arbitrary data, SQL servers can validate data they are given, and perform complex search operations and manipulation of data. 

##4.	Backups
1.	rsnapshot - Back up data using rsync and hard links. Each backup can be accessed using standard tools, so it's easy to restore data or compare the content two backups.
    
2.	Bacula - A more traditional backup system that provides better support for non-POSIX platforms like Windows.
    
3.	Dumping data for consistency - If an application is writing to a file (especially a database server) during a backup, it may not be reliable to simply copy the data file. One option is to dump the content of the file using application-specific tools that capture the data in a consistent state, while allowing the server to continue.
    
4.	Snapshots - Rather than dumping data, which can be resource-intensive for large data sets, you can instead instruct your servers to make their data consistent on disk, snapshot the disk, and then back up the snapshot. Under Windows, this is handled by the VSS service. Linux does not currently have a single consistent interface for doing this, but provides the components necessary to do so. 

##5.	Security Basics
1.	POSIX filesystem attributes - Files may have one or more owners, one or more user groups, and permissions that should apply to each of those and to all users and groups that don't have specific permissions.

2.	SELinux - On top of the POSIX standard, SELinux labels files with a type and defines additional rules for the types that users and applications can use. When access is denied by POSIX rules, no log entries are created, but when access is denied by SELinux, the system will log to its audit log file.
      
1.	audit2why - Feed audit logs to this tool to determine if there is a configuration setting that can be changed to allow access to resources denied by SELinux.
      
2.	audit2allow - When there are no configuration settings, you can use "audit2allow -M" to create a policy module that will grant access to resources which are denied under the standard policy. 

##6.	IP Networking
1.	Ethernet and switches - Learn how systems using Ethernet learn the MAC address of other systems in their broadcast domain and send packets to those systems.
    
2.	IPv4 packet transmission and routing - Learn how a host using IP creates packets, reviews its routing table to determine if the destination is directly attached or should use a gateway, and then sends the packet to the destination or gateway.
    
3.	Pv6 changes - Examine the changes between ARP and IPv6 neighbor discovery, and the address size changes in IPv6. Review SLAAC and DHCPv6. Review conventions for subnet allocation in IPv6.
    
4.	NAT - Specifically, look at the applications that break under NAT, how some protocols are worked around, and how those workarounds become difficult or impossible as we encrypt more traffic. 

##7.	Encryption Basics 

Although encryption is a very large and complex topic, you should understand at least the difference between symmetric and asymmetric (public-key) cryptography, and that data can be signed with a private key and verified with the corresponding public key, or encrypted with a public key and decrypted with the corresponding private key.

##8.	Capacity planning 

Measure resource utilization over time, and make graphs in order to determine how quickly utilization grows, so that you can estimate how long a given hardware system will be capable of handling user needs. Set up monitoring systems to alert you before your systems reach their capacity.

##9.	Troubleshooting 

Learn to record and review application operation using tools like wireshark, strace, ltrace, and gdb.

