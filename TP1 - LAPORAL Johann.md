# [TP1 : Montage RAID] - LAPORAL Johann
# 0. Prérequis
# I. Fundamentals

Lister tous les périphériques de stockage branchés à la VM
[admin.jlaporal@storage.b3 ~]fdisk -l
```bash
/dev/sda1  *        2048 39942143 39940096   19G 83 Linux  
/dev/sda2       39944190 41940991  1996802  975M  5 Extended  
/dev/sda5       39944192 41940991  1996800  975M 82 Linux swap / Solaris
```

Lister toutes les partitions des périphériques de stockage
[admin.jlaporal@storage.b3 ~]lsblk
```bash
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS  
sda      8:0    0   20G  0 disk  
├─sda1   8:1    0   19G  0 part /  
├─sda2   8:2    0    1K  0 part  
└─sda5   8:5    0  975M  0 part [SWAP]  
sdb      8:16   0   10G  0 disk  
sdc      8:32   0   10G  0 disk  
sdd      8:48   0   10G  0 disk  
sde      8:64   0   10G  0 disk  
sdf      8:80   0   10G  0 disk  
sdg      8:96   0   10G  0 disk  
sr0     11:0    1 1024M  0 rom
```

Effectuer un test SMART sur le disque
[admin.jlaporal@storage.b3 ~]sudo dnf install smartmontools  
[admin.jlaporal@storage.b3 ~]sudo systemctl status smartmontools  
[admin.jlaporal@storage.b3 ~]sudo dnf update && sudo dnf upgrade -y
[admin.jlaporal@storage.b3 ~]sudo smartctl --all /dev/sda
 
Affichez l'espace disque restant sur la partition `/`. 
[admin.jlaporal@storage.b3 ~]sudo df -h /
```bash
Filesystem      Size  Used Avail Use% Mounted on  
/dev/sda1        19G  4.7G   13G  27% /
```
 
```bash
Filesystem      Inodes  IUsed   IFree IUse% Mounted on  
/dev/sda1      1248480 162203 1086277   13% /
```
Utilisez `ioping` pour déterminer la latence du disque. 
[admin.jlaporal@storage.b3 ~]sudo ioping -c 10 /dev/sda
 
```bash
4 KiB <<< /dev/sda (block device 20 GiB): request=1 time=1.15 ms (warmup)
4 KiB <<< /dev/sda (block device 20 GiB): request=2 time=27.0 ms
4 KiB <<< /dev/sda (block device 20 GiB): request=3 time=27.7 ms
4 KiB <<< /dev/sda (block device 20 GiB): request=4 time=1.07 ms
4 KiB <<< /dev/sda (block device 20 GiB): request=5 time=28.2 ms
4 KiB <<< /dev/sda (block device 20 GiB): request=6 time=1.19 ms
4 KiB <<< /dev/sda (block device 20 GiB): request=7 time=985.1 us (fast)
4 KiB <<< /dev/sda (block device 20 GiB): request=8 time=1.21 ms (fast)
4 KiB <<< /dev/sda (block device 20 GiB): request=9 time=1.47 ms (fast)
4 KiB <<< /dev/sda (block device 20 GiB): request=10 time=531.2 us (fast)

--- /dev/sda (block device 20 GiB) ioping statistics ---
9 requests completed in 56.4 ms, 36 KiB read, 100 iops, 402.8 KiB/s
generated 10 requests in 6.00 s, 40 KiB, 1 iops, 4.44 KiB/s
min/avg/max/mdev = 531.2 us / 9.93 ms / 23.7 ms / 12.5 ms
```

Déterminer la taille du cache filesystem
[admin.jlaporal@storage.b3 ~]sudo free -h
```bash
total        used        free      shared  buff/cache   available  
Mem:           1.9Gi       1.0Gi       278Mi        11Mi       824Mi       935Mi  
Swap:          974Mi       9.1Mi       965Mi
```

# II. Partitioning
[admin.jlaporal@storage.b3 ~]sudo apt install lvm2  
[admin.jlaporal@storage.b3 ~]sudo pvcreate /dev/sdb  

```bash
physical volume "dev/sdb" successfully created.
```

Créer un Volume Group LVM nommé `storage`
[admin.jlaporal@storage.b3 ~]sudo vgcreate storage /dev/sdb
  
```bash
Volume group "storage" successfully created
```

Créer un Logical Volume LVM nommé `smol_data`
[admin.jlaporal@storage.b3 ~]sudo lvcreate -L 2G -n smol_data storage
```bash
  Logical volume "smol_data" created.
```

Créer un deuxième Logical Volume LVM nommé `big_data`
[admin.jlaporal@storage.b3 ~]sudo lvcreate -l 100%FREE -n big_data storage
```bash
  Logical volume "big_data" created.
```

Créez un système de fichiers sur les deux LVs
[admin.jlaporal@storage.b3 ~]sudo mkfs.ext4 /dev/storage/smol_data  
[admin.jlaporal@storage.b3 ~]sudo mkfs.ext4 /dev/storage/big_data

Vérifier les fichiers système : lsblk -f
```bash
  NAME                FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1              ext4        1.0            a10aab1c-4764-4ba4-8310-575651cf7cbd     12.9G    25% /
├─sda2
└─sda5              swap        1              7dd4198a-5525-45a0-aac2-90519a999a0f                  [SWAP]
sdb                 LVM2_member LVM2 001       22e2ca9c-bc19-498d-8a6e-f00ca3926893
├─storage-smol_data ext4        1.0            f3a22103-2a8a-4e78-8495-53c78524e395
└─storage-big_data  ext4        1.0            4a70a662-b2ff-4327-8bfc-d85b36ec877a
sdc
sdd
sde
sdf
sdg
sr0
```

Monter les partitions `smol_data` et `big_data`
[admin.jlaporal@storage.b3 ~]sudo mkdir -p /mnt/lvm_storage/smol  
[admin.jlaporal@storage.b3 ~]sudo mount /dev/storage/smol_data /mnt/lvm_storage/smol  

[admin.jlaporal@storage.b3 ~]sudo mkdir -p /mnt/lvm_storage/big  
[admin.jlaporal@storage.b3 ~]sudo mount /dev/storage/big_data /mnt/lvm_storage/big

```bash
/dev/mapper/storage-smol_data  2.0G   24K  1.8G   1% /mnt/lvm_storage/smol
/dev/mapper/storage-big_data   7.8G   24K  7.4G   1% /mnt/lvm_storage/big
```

# III. RAID
[admin.jlaporal@storage.b3 ~]sudo apt install mdadm
[admin.jlaporal@storage.b3 ~]sudo mdadm --version

```bash
mdadm - v4.2 - 2021-12-30 - Rocky Linux 9.5
```
# 1. Simple RAID

Mettre en place un RAID 5 avec trois disques
- Utiliser les disques `sdc`, `sdd` et `sde` pour configurer un RAID 5.  
[admin.jlaporal@storage.b3 ~]sudo apt install mdadm  
[admin.jlaporal@storage.b3 ~]sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdc /dev/sdd /dev/sde

Vérifier le RAID5 
[admin.jlaporal@storage.b3 ~]sudo mdadm --detail /dev/md0

```bash
  /dev/md0:
           Version : 1.2
     Creation Time : Sun Dec 08 12:37:45 2024
        Raid Level : raid5
        Array Size : 20953088 (19.98 GiB 21.46 GB)
     Used Dev Size : 10476544 (9.99 GiB 10.73 GB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Sun Dec 08 12:37:45 2024
             State : clean
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : storage.b3:0  (local to host storage.b3)
              UUID : e4937fd2:63af2433:53834c3e:448e82ba
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde
```

Rendre la configuration automatique au démarrage
[admin.jlaporal@storage.b3 ~]sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
[admin.jlaporal@storage.b3 ~]cat /etc/mdadm/mdadm.conf
[admin.jlaporal@storage.b3 ~]sudo update-initramfs -u
[admin.jlaporal@storage.b3 ~]sudo nano /etc/fstab

# RAID5
UUID=7156577f2-996g-454a-h65c-17f34560679b  /mnt/raid5  ext4  defaults  0  2
```

# Étape 6 : Vérification de la partition RAID
Vérifier que la partition RAID est montée correctement :
[admin.jlaporal@storage.b3 ~]df -h /mnt/raid5

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0         20G   24K   19G   1% /mnt/raid5
```

# Créer un système de fichiers sur la partition RAID

- Formater la partition RAID `/dev/md127` en ext4 avec les commandes :
[admin.jlaporal@storage.b3 ~]sudo umount /dev/md127
[admin.jlaporal@storage.b3 ~]sudo mkfs.ext4 /dev/md127


# Monter la partition RAID

1. Modifier `/etc/fstab` :
     ```bash
     UUID=6bf42840-0ac1-4a02-ae17-06b8307da6f0  /mnt/raid_storage  ext4  defaults  0  2
     ```

2. Créer le répertoire de montage :
[admin.jlaporal@storage.b3 ~]sudo mkdir -p /mnt/raid_storage

3. Démonter la partition RAID de `/mnt/raid5` :
[admin.jlaporal@storage.b3 ~]sudo umount /mnt/raid5

4. Monter la partition RAID :
[admin.jlaporal@storage.b3 ~]sudo mount -a

5. Vérifier le montage :
[admin.jlaporal@storage.b3 ~]df -h

```bash    
       Filesystem      Size  Used Avail Use% Mounted on
     /dev/md127       20G   24K   19G   1% /mnt/raid_storage
```

1. La partition est bien montée
[admin.jlaporal@storage.b3 ~]df -h
     
```bash
       Filesystem      Size  Used Avail Use% Mounted on
     /dev/md127       20G   24K   19G   1% /mnt/raid_storage
```

1. Il y a bien l'espace disponible attendu
[admin.jlaporal@storage.b3 ~]df -h /mnt/raid_storage

```bash
     Filesystem      Size  Used Avail Use% Mounted on
     /dev/md127       20G   24K   19G   1% /mnt/raid_storage
```

2. Vous pouvez lire et écrire sur la partition

# Lire sur la partition
- Créer un fichier de test dans `/mnt/raid_storage` avec la commande suivante :
[admin.jlaporal@storage.b3 ~]echo "Test" > /mnt/raid_storage/test_file.txt

- Lire le contenu du fichier avec `cat` :
```bash
  cat /mnt/raid_storage/test_file.txt
```

# Écrire sur la partition
- Ajout de contenu au fichier de test :
[admin.jlaporal@storage.b3 ~]echo "Écriture sur la partition RAID" >> /mnt/raid_storage/test_file.txt

- Lire le contenu du fichier modifié avec `cat` :
[admin.jlaporal@storage.b3 ~]cat /mnt/raid_storage/test_file.txt

Combien de Go disponibles avec un RAID 5 sur des disques de 10 Go ?
20Go pour les données et 10Go pour les checksum 

Mini benchmark
1. Effectuez un test de vitesse d'écriture sur la partition RAID monté : `/mnt/raid_storage`.

[admin.jlaporal@storage.b3 ~]sudo dd if=/dev/zero of=/mnt/raid_storage/testfile bs=1M count=1024 oflag=direct

2. Effectuez un test sur la partition LVM créée dans la partie précédente : `/mnt/lvm_storage`.
[admin.jlaporal@storage.b3 ~]sudo dd if=/dev/zero of=/mnt/lvm_storage/testfile bs=1M count=1024 oflag=direct