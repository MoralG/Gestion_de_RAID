## Ejercicio: RAID 1

----------------------------------------------------------------
Vamos a crear una máquina virtual con un sistema operativo Linux. Esta máquina va a tener conectada dos discos virtuales de 1 GB. Debemos instalar el software necesario para crear un raid 1 con dichos discos, para ello realiza los siguientes ejercicios:

* **Tarea 1:** Entrega un fichero Vagranfile donde definimos la máquina con los discos necesarios para hacer el ejercicio. Además al crear la máquina con vagrant se debe instalar el programa mdadm que nos permite la construcción del RAID.

~~~
 # -*- mode: ruby -*-
 # vi: set ft=ruby :

Vagrant.configure("2") do |config|
    disco1 = ".vagrant/disco1.vdi" 
    disco2 = ".vagrant/disco2.vdi"  

    config.vm.box = "debian/buster64" 
    config.vm.hostname = "Raid" 
    config.vm.network :public_network,:bridge=>"wlp2s0"   
    config.vm.provider :virtualbox do |v|
        if not File.exist?(disco1)
                v.customize ["createhd", "--filename", disco1, "--size", 1 *  1024]
            v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1,"--device", 0, "--type", "hdd", "--medium", disco1]
        end

            if not File.exist?(disco2)
                  v.customize ["createhd", "--filename", disco2, "--size", 1 * 1024]
            v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 2,"--device", 0, "--type", "hdd", "--medium", disco2]
            end
      end
end
~~~

* **Tarea 2:** Crea una raid llamado md1 con los dos discos que hemos conectado a la máquina.

~~~
sudo apt install mdadm
   Reading package lists... Done
   Building dependency tree       
   Reading state information... Done
   mdadm is already the newest version (4.1-1).
   0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
   mdadm: /dev/sdb appears to be part of a raid array:
          level=raid1 devices=2 ctime=Fri Sep 27 18:39:05 2019
   mdadm: partition table exists on /dev/sdb but will be lost or
          meaningless after creating array
   mdadm: Note: this array has metadata at the start and
       may not be suitable as a boot device.  If you plan to
       store '/boot' on this device please ensure that
       your boot-loader understands md/v1.x metadata, or use
       --metadata=0.90
   mdadm: /dev/sdc appears to be part of a raid array:
          level=raid1 devices=2 ctime=Fri Sep 27 18:39:05 2019
   Continue creating array? 
   Continue creating array? (y/n) y
   mdadm: Defaulting to version 1.2 metadata
   mdadm: array /dev/md0 started.
~~~

* **Tarea 3:** Comprueba las características del RAID. Comprueba el estado del RAID. ¿Qué capacidad tiene el RAID que hemos creado?

~~~
lsblk -f
   NAME   FSTYPE            LABEL  UUID                                 FSAVAIL FSUSE%    MOUNTPOINT
   sda                                                                                 
   ├─sda1 ext4                     b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5     16G     8% /
   ├─sda2                                                                              
   └─sda5 swap                     f8f6d279-1b63-4310-a668-cb468c9091d8                   [SWAP]
   sdb    linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                               
   sdc    linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                               
sudo mdadm --detail /dev/md0
   /dev/md0:
              Version : 1.2
        Creation Time : Fri Sep 27 18:46:41 2019
           Raid Level : raid1
           Array Size : 1046528 (1022.00 MiB 1071.64 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 2
        Total Devices : 2
          Persistence : Superblock is persistent
   
          Update Time : Fri Sep 27 18:46:47 2019
                State : clean 
       Active Devices : 2
      Working Devices : 2
       Failed Devices : 0
        Spare Devices : 0
   
   Consistency Policy : resync
   
                 Name : Raid:0  (local to host Raid)
                 UUID : 94aae06d:584491b9:f120091b:250cd582
               Events : 17
   
       Number   Major   Minor   RaidDevice State
          0       8       16        0      active sync   /dev/sdb
          1       8       32        1      active sync   /dev/sdc
~~~

* **Tarea 4:** Crea una partición primaria de 500Mb en el raid1.

~~~
sudo fdisk /dev/md0

   Welcome to fdisk (util-linux 2.33.1).
   Changes will remain in memory only, until you decide to write them.
   Be careful before using the write command.
   
   Device does not contain a recognized partition table.
   Created a new DOS disklabel with disk identifier 0x4a19c8e8.
   
   Command (m for help): n
   Partition type
      p   primary (0 primary, 0 extended, 4 free)
      e   extended (container for logical partitions)
   Select (default p): p
   Partition number (1-4, default 1): 1
   First sector (2048-2093055, default 2048): 
   Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2093055, default 2093055): +500M
   
   Created a new partition 1 of type 'Linux' and of size 500 MiB.
   
   Command (m for help): p
   Disk /dev/md0: 1022 MiB, 1071644672 bytes, 2093056 sectors
   Units: sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disklabel type: dos
   Disk identifier: 0x4a19c8e8
   
   Device     Boot Start     End Sectors  Size Id Type
   /dev/md0p1       2048 1026047 1024000  500M 83 Linux
   
   Command (m for help): w
   The partition table has been altered.
   Calling ioctl() to re-read partition table.
   Syncing disks.
~~~

* **Tarea 5:** Formatea esa partición con un sistema de archivo ext3.

~~~
lsblk -f
   NAME      FSTYPE            LABEL  UUID                                 FSAVAIL FSUSE%    MOUNTPOINT
   sda                                                                                    
   ├─sda1    ext4                     b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5     16G     8%    /
   ├─sda2                                                                                 
   └─sda5    swap                     f8f6d279-1b63-4310-a668-cb468c9091d8                   [SWAP]
   sdb       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                                  
     └─md0p1                                                                              
   sdc       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                                  
     └─md0p1                                                                              

sudo mkfs.ext3 /dev/md0p1 
   mke2fs 1.44.5 (15-Dec-2018)
   Creating filesystem with 512000 1k blocks and 128016 inodes
   Filesystem UUID: 5d58988a-86fd-46dd-ad2d-0efe93628ee5
   Superblock backups stored on blocks: 
       8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409
   
   Allocating group tables: done                            
   Writing inode tables: done                            
   Creating journal (8192 blocks): done
   Writing superblocks and filesystem accounting information: done 

lsblk -f
   NAME      FSTYPE            LABEL  UUID                                 FSAVAIL FSUSE%    MOUNTPOINT
   sda                                                                                    
   ├─sda1    ext4                     b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5     16G     8%    /
   ├─sda2                                                                                 
   └─sda5    swap                     f8f6d279-1b63-4310-a668-cb468c9091d8                   [SWAP]
   sdb       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                                  
     └─md0p1 ext3                     5d58988a-86fd-46dd-ad2d-0efe93628ee5                
   sdc       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                                  
     └─md0p1 ext3                     5d58988a-86fd-46dd-ad2d-0efe93628ee5  
~~~

* **Tarea 6:** Monta la partición en el directorio /mnt/raid1 y crea un fichero. ¿Qué tendríamos que hacer para que este punto de montaje sea permanente?

~~~
sudo mkdir /mnt/raid0

sudo mount /dev/md0p1 /mnt/raid0

sudo touch /mnt/raid0/prueba.txt

lsblk -f
  NAME      FSTYPE            LABEL  UUID                                 FSAVAIL FSUSE%   MOUNTPOINT
  sda                                                                                    
  ├─sda1    ext4                     b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5     16G     8%   /
  ├─sda2                                                                                 
  └─sda5    swap                     f8f6d279-1b63-4310-a668-cb468c9091d8                  [SWAP]
  sdb       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
  └─md0                                                                                  
    └─md0p1 ext3                     5d58988a-86fd-46dd-ad2d-0efe93628ee5  448.9M     0%   /mnt/raid0
  sdc       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
  └─md0                                                                                  
    └─md0p1 ext3                     5d58988a-86fd-46dd-ad2d-0efe93628ee5  448.9M     0%   /mnt/raid0
~~~

* **Tarea 7:** Simula que un disco se estropea. Muestra el estado del raid para comprobar que un disco falla. ¿Podemos acceder al fichero?

~~~
sudo mdadm /dev/md0 -f /dev/sdb
   mdadm: set /dev/sdb faulty in /dev/md0

sudo mdadm --detail /dev/md0
   /dev/md0:
              Version : 1.2
        Creation Time : Fri Sep 27 18:46:41 2019
           Raid Level : raid1
           Array Size : 1046528 (1022.00 MiB 1071.64 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 2
        Total Devices : 2
          Persistence : Superblock is persistent
   
          Update Time : Fri Sep 27 19:13:57 2019
                State : clean, degraded 
       Active Devices : 1
      Working Devices : 1
       Failed Devices : 1
        Spare Devices : 0
   
   Consistency Policy : resync
   
                 Name : Raid:0  (local to host Raid)
                 UUID : 94aae06d:584491b9:f120091b:250cd582
               Events : 19
   
       Number   Major   Minor   RaidDevice State
          -       0        0        0      removed
          1       8       32        1      active sync   /dev/sdc
   
          0       8       16        -      faulty   /dev/sdb
   You have new mail in /var/mail/vagrant

ls /mnt/raid0/
   lost+found  prueba.txt
~~~

* **Tarea 8:** Recupera el estado del raid y comprueba si podemos acceder al fichero.

~~~
sudo mdadm /dev/md0 --remove /dev/sdb
   mdadm: hot removed /dev/sdb from /dev/md0

lsblk -f
   NAME      FSTYPE            LABEL  UUID                                 FSAVAIL FSUSE%    MOUNTPOINT
   sda                                                                                    
   ├─sda1    ext4                     b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5     16G     8%    /
   ├─sda2                                                                                 
   └─sda5    swap                     f8f6d279-1b63-4310-a668-cb468c9091d8                   [SWAP]
   sdb       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   sdc       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                                  
     └─md0p1 ext3                     5d58988a-86fd-46dd-ad2d-0efe93628ee5  448.9M     0%    /mnt/raid0

sudo mdadm /dev/md0 --add /dev/sdb
   mdadm: added /dev/sdb

lsblk -f
   NAME      FSTYPE            LABEL  UUID                                 FSAVAIL FSUSE%    MOUNTPOINT
   sda                                                                                    
   ├─sda1    ext4                     b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5     16G     8%    /
   ├─sda2                                                                                 
   └─sda5    swap                     f8f6d279-1b63-4310-a668-cb468c9091d8                   [SWAP]
   sdb       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                                  
     └─md0p1 ext3                     5d58988a-86fd-46dd-ad2d-0efe93628ee5  448.9M     0%    /mnt/raid0
   sdc       linux_raid_member Raid:0 94aae06d-5844-91b9-f120-091b250cd582                
   └─md0                                                                                  
     └─md0p1 ext3                     5d58988a-86fd-46dd-ad2d-0efe93628ee5  448.9M     0%    /mnt/raid0

ls /mnt/raid0/
   lost+found  prueba.txt
~~~

* **Tarea 9:** ¿Se puede añadir un nuevo disco al raid 1?. Compruebalo.

Si se puede, pero tenemos que añadirlo en el fichero Vagrantfile y luego tenemos que añadirlo al md0.

~~~
 # -*- mode: ruby -*-
 # vi: set ft=ruby :

Vagrant.configure("2") do |config|
    disco1 = ".vagrant/disco1.vdi" 
    disco2 = ".vagrant/disco2.vdi"  
    disco3 = ".vagrant/disco3.vdi" 

    config.vm.box = "debian/buster64" 
    config.vm.hostname = "Raid" 
    config.vm.network :public_network,:bridge=>"wlp2s0"   
    config.vm.provider :virtualbox do |v|
        if not File.exist?(disco1)
                v.customize ["createhd", "--filename", disco1, "--size", 1 *  1024]
            v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1,"--device", 0, "--type", "hdd", "--medium", disco1]
        end

            if not File.exist?(disco2)
                  v.customize ["createhd", "--filename", disco2, "--size", 1 * 1024]
            v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 2,"--device", 0, "--type", "hdd", "--medium", disco2]
            end

        if not File.exist?(disco3)
                  v.customize ["createhd", "--filename", disco3, "--size", 1 * 1024]
            v.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 3,"--device", 0, "--type", "hdd", "--medium", disco3]
            end
      end
end
~~~

~~~
sudo mdadm --grow /dev/md0 --raid-devices=3 --add /dev/sdd
   mdadm: added /dev/sdd
   raid_disks for /dev/md0 set to 3

sudo mdadm --detail /dev/md0
   /dev/md0:
              Version : 1.2
        Creation Time : Mon Sep 30 10:24:52 2019
           Raid Level : raid1
           Array Size : 1046528 (1022.00 MiB 1071.64 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 3
          Persistence : Superblock is persistent
   
          Update Time : Mon Sep 30 10:33:40 2019
                State : clean 
       Active Devices : 3
      Working Devices : 3
       Failed Devices : 0
        Spare Devices : 0
   
   Consistency Policy : resync
   
                 Name : Raid:0  (local to host Raid)
                 UUID : 5d18f51f:f911fdc7:d0673b3a:9c7d60e0
               Events : 40
   
       Number   Major   Minor   RaidDevice State
          0       8       17        0      active sync   /dev/sdb
          1       8       33        1      active sync   /dev/sdc
          2       8       49        2      active sync   /dev/sdd
~~~

* **Tarea 10:** Redimensiona la partición y el sistema de archivo de 500Mb al tamaño del raid.

~~~
sudo fdisk /dev/md0 

   Welcome to fdisk (util-linux 2.33.1).
   Changes will remain in memory only, until you decide to write them.
   Be careful before using the write command.
   
   Command (m for help): d
   Selected partition 1
   Partition 1 has been deleted.
   
   Command (m for help): n
   Partition type
      p   primary (0 primary, 0 extended, 4 free)
      e   extended (container for logical partitions)
   Select (default p): p
   Partition number (1-4, default 1): 1
   First sector (2048-2093055, default 2048): 
   Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2093055, default 2093055): 
   
   Created a new partition 1 of type 'Linux' and of size 1021 MiB.
   Partition #1 contains a ext3 signature.
   
   Do you want to remove the signature? [Y]es/[N]o: y
   
   The signature will be removed by a write command.
   
   Command (m for help): w
   The partition table has been altered.
   Syncing disks.
   
lsblk 
   NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   sda           8:0    0 19.8G  0 disk  
   ├─sda1        8:1    0 18.8G  0 part  /
   ├─sda2        8:2    0    1K  0 part  
   └─sda5        8:5    0 1021M  0 part  [SWAP]
   sdb           8:16   0    1G  0 disk  
   └─sdb1        8:17   0 1023M  0 part  
     └─md0       9:0    0 1022M  0 raid1 
       └─md0p1 259:1    0 1021M  0 part  /mnt/raid0
   sdc           8:32   0    1G  0 disk  
   └─sdc1        8:33   0 1023M  0 part  
     └─md0       9:0    0 1022M  0 raid1 
       └─md0p1 259:1    0 1021M  0 part  /mnt/raid0
   sdd           8:48   0    1G  0 disk  
   └─sdd1        8:49   0 1023M  0 part  
     └─md0       9:0    0 1022M  0 raid1 
       └─md0p1 259:1    0 1021M  0 part  /mnt/raid0

sudo resize2fs  /dev/md0p1
   resize2fs 1.44.5 (15-Dec-2018)
   Filesystem at /dev/md0p1 is mounted on /mnt/raid0; on-line resizing required
   old_desc_blocks = 2, new_desc_blocks = 4
   The filesystem on /dev/md0p1 is now 1045504 (1k) blocks long.
~~~
