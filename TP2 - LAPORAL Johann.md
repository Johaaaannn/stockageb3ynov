# [TP2 : Modest Storage SAN] - LAPORAL Johann
# [II. Partie 2 : SAN network et iSCSI]
# Disks and RAID 

Configurer des RAIDs

[admin.jlaporal@sto1 ~]# cat /proc/mdstat
```bash
Personalities : [raid1] 
md2 : active raid1 sdg[1] sdb[0]
  1046528 blocks super 1.2 [2/2] [UU]
      
md1 : active raid1 sde[1] sdf[0]
      1046528 blocks super 1.2 [2/2] [UU]
      
md0 : active raid1 sdc[1] sdd[0]
      1046528 blocks super 1.2 [2/2] [UU]
```

[admin.jlaporal@sto2 ~]# cat /proc/mdstat
```bash
Personalities : [raid1] 
md2 : active raid1 sdg[1] sdf[0]
      1046528 blocks super 1.2 [2/2] [UU]
      
md1 : active raid1 sde[1] sdd[0]
      1046528 blocks super 1.2 [2/2] [UU]
      
md0 : active raid1 sdc[1] sdb[0]
      1046528 blocks super 1.2 [2/2] [UU]
```


Prouvez que vous avez 3 volumes RAID prêts à l'emploi

[admin.jlaporal@sto2-1 ~]# lsblk
```bash
NAME             MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                8:0    0   20G  0 disk  
├─sda1             8:1    0    1G  0 part  /boot
└─sda2             8:2    0   19G  0 part  
sdb                8:16   0    1G  0 disk  
└─md0              9:0    0 1022M  0 raid1 /mnt/raid1
sdc                8:32   0    1G  0 disk  
└─md0              9:0    0 1022M  0 raid1 /mnt/raid1
sdd                8:48   0    1G  0 disk  
└─md1              9:1    0 1022M  0 raid1 /mnt/raid2
sde                8:64   0    1G  0 disk  
└─md1              9:1    0 1022M  0 raid1 /mnt/raid2
sdf                8:80   0    1G  0 disk  
└─md2              9:2    0 1022M  0 raid1 /mnt/raid3
sdg                8:96   0    1G  0 disk  
└─md2              9:2    0 1022M  0 raid1 /mnt/raid3
```

# B. iSCSI target

Installer target
[admin.jlaporal@sto1 ~]#sudo dnf install targetcli

Démarrer le service installer
[admin.jlaporal@sto2 ~]#sudo systemctl start target

```bash
Configurer les targets iSCSI
/> /backstores/fileio 
/backstores/fileio> create name=data-chunk2 file_or_dev=/dev/md1
Note: block backstore preferred for best results
Created fileio data-chunk2 with size 1071644672
exit

/> ls
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 3]
  | | o- data-chunk1 ................................................................... [/dev/md0 (1022.0MiB) write-back activated]
  | | | o- alua ................................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | | o- data-chunk2 ................................................................... [/dev/md1 (1022.0MiB) write-back activated]
  | | | o- alua ................................................................................................... [ALUA Groups: 1]
  | | |   o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | | o- data-chunk3 ................................................................... [/dev/md2 (1022.0MiB) write-back activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 3]
  | o- iqn.2024-12.tp2.b3:data-chunk1 .................................................................................... [TPGs: 1]
  | | o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  | |   o- acls .......................................................................................................... [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator ...................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk1 (rw)]
  | |   o- luns .......................................................................................................... [LUNs: 1]
  | |   | o- lun0 ............................................................... [fileio/data-chunk1 (/dev/md0) (default_tg_pt_gp)]
  | |   o- portals .................................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ..................................................................................................... [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk2 .................................................................................... [TPGs: 1]
  | | o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  | |   o- acls .......................................................................................................... [ACLs: 1]
  | |   | o- iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator ...................................................... [Mapped LUNs: 1]
  | |   |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk2 (rw)]
  | |   o- luns .......................................................................................................... [LUNs: 1]
  | |   | o- lun0 ............................................................... [fileio/data-chunk2 (/dev/md1) (default_tg_pt_gp)]
  | |   o- portals .................................................................................................... [Portals: 1]
  | |     o- 0.0.0.0:3260 ..................................................................................................... [OK]
  | o- iqn.2024-12.tp2.b3:data-chunk3 .................................................................................... [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 1]
  |     | o- iqn.2024-12.tp2.b3:data-chunk3:chunk1-initiator ...................................................... [Mapped LUNs: 1]
  |     |   o- mapped_lun0 .......................................................................... [lun0 fileio/data-chunk3 (rw)]
  |     o- luns .......................................................................................................... [LUNs: 1]
  |     | o- lun0 ............................................................... [fileio/data-chunk3 (/dev/md2) (default_tg_pt_gp)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ..................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
/> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup/.
Configuration saved to /etc/target/saveconfig.json

/> /iscsi 
/iscsi> create iqn.2024-12.tp2.b3:data-chunk1
Created target iqn.2024-12.tp2.b3:data-chunk1.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
```

# 2. Chunks machine
# A. Simple iSCSI

Installer les tools iSCSI sur chunk1.tp2.b3
[admin.jlaporal@sto2 ~]# sudo dnf install iscsi-initiator-utils -y


Configurer un iSCSI initiator
[admin.jlaporal@chunk3 ~]# sudo iscsiadm -m discoverydb -t st -p 10.3.1.1:3260 --discover
```bash
sudo iscsiadm -m discoverydb -t st -p 10.3.1.2:3260 --discover
sudo iscsiadm -m discoverydb -t st -p 10.3.2.1:3260 --discover
sudo iscsiadm -m discoverydb -t st -p 10.3.2.2:3260 --discover
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.1.1:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.1.2:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.2.1:3260,1 iqn.2024-12.tp2.b3:data-chunk3
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk1
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk2
10.3.2.2:3260,1 iqn.2024-12.tp2.b3:data-chunk3
```

[admin.jlaporal@chunk1 ~]# cat /etc/iscsi/initiatorname.iscsi 
```bash
InitiatorName=iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator

sudo iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.1.1 --login
sudo iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.1.2 --login
sudo iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.2.1 --login
sudo iscsiadm -m node -T iqn.2024-12.tp2.b3:data-chunk1 -p 10.3.2.2 --login
```

Modifier la configuration du démon iSCSI
[admin.jlaporal@chunk1 ~]# cat /etc/iscsi/initiatorname.iscsi 
```bash
InitiatorName=iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
```

Prouvez que la configuration est prête
[admin.jlaporal@chunk1 ~]# lsblk
```bash
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                8:0    0   20G  0 disk 
├─sda1             8:1    0    1G  0 part /boot
└─sda2             8:2    0   19G  0 part
sdb                8:16   0 1022M  0 disk 
sdc                8:32   0 1022M  0 disk 
sdd                8:48   0 1022M  0 disk 
sde                8:64   0 1022M  0 disk 
sr0               11:0    1 1024M  0 rom  
```

[admin.jlaporal@chunk1 ~]#sudo iscsiadm -m session -P 3
```bash
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk1 (non-flash)
	Current Portal: 10.3.2.1:3260,1
	Persistent Portal: 10.3.2.1:3260,1
        [...]
	Current Portal: 10.3.1.1:3260,1
	Persistent Portal: 10.3.1.1:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
		Iface IPaddress: 10.3.1.101
		Iface HWaddress: default
		Iface Netdev: default
		[...]
	Current Portal: 10.3.1.2:3260,1
	Persistent Portal: 10.3.1.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
		Iface IPaddress: 10.3.1.101
		Iface HWaddress: default
		Iface Netdev: default
		SID: 7
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
        [...]
	Current Portal: 10.3.2.2:3260,1
	Persistent Portal: 10.3.2.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk1:chunk1-initiator
		Iface IPaddress: 10.3.2.101
		Iface HWaddress: default
		Iface Netdev: default
		SID: 8
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 120
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
```


Installer les outils multipath sur `chunk1.tp2.b3`
[admin.jlaporal@chunk1 ~]#sudo dnf install device-mapper-multipath -y
```bash
=============================================
[...]
Upgraded:
  device-mapper-9:1.02.198-2.el9.x86_64      
  device-mapper-event-9:1.02.198-2.el9.x86_64
  device-mapper-event-libs-9:1.02.198-2.el9.x86_64
  device-mapper-libs-9:1.02.198-2.el9.x86_64 
  kpartx-0.8.7-32.el9.x86_64                 
  lvm2-9:2.03.24-2.el9.x86_64                
  lvm2-libs-9:2.03.24-2.el9.x86_64           
Installed:
  device-mapper-multipath-0.8.7-32.el9.x86_64
  device-mapper-multipath-libs-0.8.7-32.el9.x86_64
```

Configurer le fichier `/etc/multipath.conf`
[admin.jlaporal@chunk1 etc]# cat /etc/multipath.conf | head -n 10
```bash
blacklist_exceptions {
	device {
		vendor	"IBM"
		product	"S/390.*"
	}
}

defaults {
  user_friendly_names yes
  find_multipaths yes
```

Démarrer le service `multipathd`
[admin.jlaporal@chunk1 etc]#sudo systemctl status multipathd
```bash
● multipathd.service - Device-Mapper Multipath Device Controller
     Loaded: loaded (/usr/lib/systemd/system/multipathd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2024-12-09 19:45:45 CET;
TriggeredBy: ○ multipathd.socket
    Process: 2053 ExecStartPre=/sbin/modprobe -a scsi_dh_alua scsi_dh_emc scsi_dh_rdac dm-multipath (code=exited, status=0/SUCCESS)
    Process: 2057 ExecStartPre=/sbin/multipath -A (code=exited, status=0/SUCCESS)
   Main PID: 2058 (multipathd)
     Status: "up"
      Tasks: 7
     Memory: 19.5M
        CPU: 90ms
     CGroup: /system.slice/multipathd.service
             └─2058 /sbin/multipathd -d -s
```

# 3. Formatage et montage
Créez une partition sur les devices `mpatha` et `mpathb`
[admin.jlaporal@chunk1 etc]# fdisk /dev/mapper/mpatha 

```bash
Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xd036d41a.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (16384-2093055, default 16384): w
Value out of range.
First sector (16384-2093055, default 16384): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (16384-2093055, default 2093055): 

Created a new partition 1 of type 'Linux' and of size 1014 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Re-reading the partition table failed.: Invalid argument

The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or partx(8).
```

Formatez en `xfs` les partitions
[admin.jlaporal@chunk1 ~]# lsblk -f
```bash
NAME             FSTYPE       FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
[...]
sdb              mpath_member                                                                      
└─mpatha                                                                                           
  └─mpatha1      xfs                         13d2266c-66c5-4fc0-bae3-893a5c48c25d                  
sdc              mpath_member                                                                      
└─mpatha                                                                                           
  └─mpatha1      xfs                         13d2266c-66c5-4fc0-bae3-893a5c48c25d                  
sdd              mpath_member                                                                      
└─mpathb                                                                                           
  └─mpathb1      xfs                         497683c6-4058-49c9-b324-e353538467fe                  
sde              mpath_member                                                                      
└─mpathb                                                                                           
  └─mpathb1      xfs                         497683c6-4058-49c9-b324-e353538467fe 
```

Point de montage `/mnt/data_chunk1`
[admin.jlaporal@chunk2 ~]# cat /etc/systemd/system/mnt-data_chunk1.mount 
```bash
[Unit]
Description=Mount for /mnt/data_chunk1
After=network-online.target
Wants=network-online.target

[Mount]
What=/dev/mapper/mpatha
Where=/mnt/data_chunk1
Type=xfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

[admin.jlaporal@chunk2 ~]# lsblk -f
```bash
NAME             FSTYPE       FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
[...]
sdd              mpath_member                                                                      
└─mpatha                                                                                           
  └─mpatha1      xfs                         8fcba48c-4af9-4078-89a7-a0b9df411301    911.1M     4% /mnt/data_chunk1
sde              mpath_member                                                                      
└─mpatha                                                                                           
  └─mpatha1      xfs                         8fcba48c-4af9-4078-89a7-a0b9df411301    911.1M     4% /mnt/data_chunk1
sr0
[admin.jlaporal@chunk2 ~]# df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/mapper/mpatha1       950M   39M  912M   5% /mnt/data_chunk1
```

# 4. Tests

# A. Simulation de panne

Simuler une coupure réseau
[admin.jlaporal@chunk2 ~]# date
```bash
Mon Dec  9 19:53:37 CET 2024

mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=active
| `- 5:0:0:0 sdd 8:48 failed faulty running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=enabled
| `- 7:0:0:0 sdb 8:16 failed faulty running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running
```

# B. Jouer avec les paramètres

Resimuler une panne

[admin.jlaporal@chunk2 ~]#date
```bash
Mon Dec  9 19:59:12 CET 2024

mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=active
| `- 5:0:0:0 sdd 8:48 failed faulty running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=0 status=enabled
| `- 7:0:0:0 sdb 8:16 failed faulty running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running
```

# 5. Replicate

Preuve du setup

[admin.jlaporal@chunk2 ~]# lsblk
```bash
NAME             MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                8:0    0   20G  0 disk  
├─sda1             8:1    0    1G  0 part  /boot
└─sda2             8:2    0   19G  0 part  
sdb                8:16   0 1022M  0 disk  
└─mpathb         253:3    0 1022M  0 mpath 
  └─mpathb1      253:5    0 1014M  0 part  /mnt/data_chunk2
sdc                8:32   0 1022M  0 disk  
└─mpathb         253:3    0 1022M  0 mpath 
  └─mpathb1      253:5    0 1014M  0 part  /mnt/data_chunk2
sdd                8:48   0 1022M  0 disk  
└─mpatha         253:2    0 1022M  0 mpath 
  └─mpatha1      253:4    0 1014M  0 part  /mnt/data_chunk1
sde                8:64   0 1022M  0 disk  
└─mpatha         253:2    0 1022M  0 mpath 
  └─mpatha1      253:4    0 1014M  0 part  /mnt/data_chunk1
sr0               11:0    1 1024M  0 rom   
```

[admin.jlaporal@chunk2 ~]# df -h
```bash
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     882M     0  882M   0% /dev/shm
tmpfs                     353M  5.0M  348M   2% /run
/dev/mapper/rl_vbox-admin.jlaporal   17G  1.2G   16G   8% /
/dev/sda1                 960M  223M  738M  24% /boot
tmpfs                     177M     0  177M   0% /run/user/0
/dev/mapper/mpatha1       950M   39M  912M   5% /mnt/data_chunk1
/dev/mapper/mpathb1       950M   42M  909M   5% /mnt/data_chunk2
```

[admin.jlaporal@chunk2 ~]#sudo iscsiadm -m session -P 3
```bash
iSCSI Transport Class version 2.0-870
version 6.2.1.9
Target: iqn.2024-12.tp2.b3:data-chunk2 (non-flash)
	Current Portal: 10.3.2.2:3260,1
	Persistent Portal: 10.3.2.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.2.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 10
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 6	State: running
		scsi6 Channel 00 Id 0 Lun: 0
			Attached scsi disk sde		State: running
	Current Portal: 10.3.1.1:3260,1
	Persistent Portal: 10.3.1.1:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.1.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 5
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 7	State: running
		scsi7 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdb		State: running
	Current Portal: 10.3.2.1:3260,1
	Persistent Portal: 10.3.2.1:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.2.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 6
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 8	State: running
		scsi8 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdc		State: running
	Current Portal: 10.3.1.2:3260,1
	Persistent Portal: 10.3.1.2:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.2024-12.tp2.b3:data-chunk2:chunk1-initiator
		Iface IPaddress: 10.3.1.102
		Iface HWaddress: default
		Iface Netdev: default
		SID: 9
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 5
		Target Reset Timeout: 30
		LUN Reset Timeout: 30
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: <empty>
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 262144
		FirstBurstLength: 65536
		MaxBurstLength: 262144
		ImmediateData: Yes
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 5	State: running
		scsi5 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdd		State: running
```

```bash
[admin.jlaporal@chunk2 ~]#sudo multipath -ll
6996.373607 | /etc/multipath.conf line 26, invalid keyword in the multipath section: path_checker
mpatha (3600140557c66dd047214999a651eb037) dm-2 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| `- 5:0:0:0 sdd 8:48 active ready running
`-+- policy='service-time 0' prio=50 status=enabled
  `- 6:0:0:0 sde 8:64 active ready running
mpathb (3600140532c8eec0828442df8df844bb1) dm-3 LIO-ORG,data-chunk2
size=1022M features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
|-+- policy='service-time 0' prio=50 status=enabled
| `- 7:0:0:0 sdb 8:16 active ready running
`-+- policy='service-time 0' prio=50 status=active
  `- 8:0:0:0 sdc 8:32 active ready running
```

# III. Distributed filesystem

# 1. Master server

Installer les paquets nécessaires pour avoir un Moose Master
[admin.jlaporal@master ~]# sudo yum install moosefs-master moosefs-cgi moosefs-cgiserv moosefs-cli

[admin.jlaporal@master ~]# sudo systemctl enable moosefs-cgiserv
```bash
Created symlink /etc/systemd/system/multi-user.target.wants/moosefs-cgiserv.service → /usr/lib/systemd/system/moosefs-cgiserv.service.
```

[admin.jlaporal@master ~]# sudo systemctl start moosefs-cgiserv

Ouvrez les ports firewall
[admin.jlaporal@master ~]#sudo firewall-cmd --list-all | grep tcp
```bash
  ports: 21/ftp 22/tcp 80/tcp 9425/tcp 9419/tcp 9420/tcp 9421/tcp
```

# 2. Chunk servers
Installer les paquets nécessaires pour avoir un Moose Chunk Server
[admin.jlaporal@chunk1 ~]#sudo yum install moosefs-chunkserver
```bash
Package moosefs-chunkserver-3.0.118-1.rhsystemd.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

Modifier la conf du Chunk Server
[admin.jlaporal@chunk1 ~]# cat /etc/hosts
```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.3.250.1  master.tp2.b3
```

[admin.jlaporal@chunk1 ~]# cat /etc/mfs/mfschunkserver.cfg | grep HOST
```bash
# BIND_HOST = *
MASTER_HOST = master.tp2.b3
# CSSERV_LISTEN_HOST = *
```

Faire appartenir les partitions à partager à l'utilisateur `mfs`
[admin.jlaporal@chunk1 ~]# ls -al /mnt/
```bash
total 24
drwxr-xr-x.   4 admin.jlaporal admin.jlaporal   44 Dec  10 19:10 .
dr-xr-xr-x.  18 admin.jlaporal admin.jlaporal  255 Dec  9 19:05 ..
drwxr-xr-x  258 mfs  mfs  8192 Dec  10 19:54 data_chunk1
drwxr-xr-x  258 mfs  mfs  8192 Dec  10 19:54 data_chunk2
```

Modifier la conf des disques du Chunk Server
[admin.jlaporal@chunk1 ~]# cat /etc/mfs/mfshdd.cfg | grep mnt/data
```bash
/mnt/data_chunk1
/mnt/data_chunk2
```

Démarrez les services du Moose Chunk Server
[admin.jlaporal@chunk1 ~]#sudo systemctl start moosefs-chunkserver
```bash
● moosefs-master.service - MooseFS Master server
     Loaded: loaded (/usr/lib/systemd/system/moosefs-master.service; enabled; preset: disabled)
     Active: active (running) since Tue 2024-12-10 20:01:36 CET;
    Process: 2240 ExecStart=/usr/sbin/mfsmaster start (code=exited, status=0/SUCCESS)
   Main PID: 2242 (mfsmaster)
      Tasks: 2 (limit: 11025)
     Memory: 216.6M
        CPU: 9.330s
     CGroup: /system.slice/moosefs-master.service
             ├─2242 /usr/sbin/mfsmaster start
             └─2243 "mfsmaster (data writer)"

Dec 10 20:05:03 master.tp2.b3 mfsmaster[2242]: chunkserver register begin (packet version: 6) - ip: 10.3.250.11 / port: 9422, usedspace: 0 (0.00 GiB), totalspace: 0 (0.00 GiB)
Dec 10 20:05:03 master.tp2.b3 mfsmaster[2242]: connection with CS(10.3.250.11) has been closed by peer
Dec 10 20:05:03 master.tp2.b3 mfsmaster[2242]: chunkserver disconnected - ip: 10.3.250.11 / port: 9422, usedspace: 0 (0.00 GiB), totalspace: 0 (0.00 GiB)
Dec 10 20:05:03 master.tp2.b3 mfsmaster[2242]: server ip: 10.3.250.11 / port: 9422 has been fully removed from data structures
Dec 10 20:06:25 master.tp2.b3 mfsmaster[2242]: chunkserver register begin (packet version: 6) - ip: 10.3.250.11 / port: 9422, usedspace: 0 (0.00 GiB), totalspace: 0 (0.00 GiB)
Dec 10 20:06:25 master.tp2.b3 mfsmaster[2242]: connection with CS(10.3.250.11) has been closed by peer
Dec 10 20:06:25 master.tp2.b3 mfsmaster[2242]: chunkserver disconnected - ip: 10.3.250.11 / port: 9422, usedspace: 0 (0.00 GiB), totalspace: 0 (0.00 GiB)
Dec 10 20:06:25 master.tp2.b3 mfsmaster[2242]: server ip: 10.3.250.11 / port: 9422 has been fully removed from data structures
Dec 10 20:06:56 master.tp2.b3 mfsmaster[2242]: chunkserver register begin (packet version: 6) - ip: 10.3.250.11 / port: 9422, usedspace: 0 (0.00 GiB), totalspace: 0 (0.00 GiB)
Dec 10 20:06:56 master.tp2.b3 mfsmaster[2242]: chunkserver register end (packet version: 6) - ip: 10.3.250.11 / port: 9422
```

# 3. Consume
# 1. Monter la partition Moose

Installer les paquets nécessaires pour avoir un Moose Client
[admin.jlaporal@web ~]#sudo yum install moosefs-client
```bash
Package moosefs-client-3.0.118-1.rhsystemd.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

Monter la partition Moose
[admin.jlaporal@web ~]#sudo mfsmount /mnt/www/ -H master.tp2.b3
```bash
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; admin.jlaporal mapped to admin.jlaporal:admin.jlaporal
```

# 2. NGINX

Installer et configurer NGINX sur la machine `web.tp2.b3`
[admin.jlaporal@web]#sudo dnf install nginx
```bash
Package nginx-2:1.20.1-20.el9.0.1.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

Démarrer le service NGINX
[admin.jlaporal@web]#sudo systemctl status nginx
```bash
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Tue 2024-12-10 20:34:34 CET;
    Process: 1332 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1333 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1334 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1335 (nginx)
      Tasks: 2 (limit: 11025)
     Memory: 2.0M
        CPU: 47ms
     CGroup: /system.slice/nginx.service
             ├─1335 "nginx: master process /usr/sbin/nginx"
             └─1336 "nginx: worker process"

Dec 10 20:34:34 web.tp2.b3 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 10 20:34:34 web.tp2.b3 nginx[1333]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 10 20:34:34 web.tp2.b3 nginx[1333]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 10 20:34:34 web.tp2.b3 systemd[1]: Started The nginx HTTP and reverse proxy server.

```