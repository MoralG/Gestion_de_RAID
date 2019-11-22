## Ejercicio: RAID 5

----------------------------------------------------------------
A continuación queremos crear un raid 5 en una máquina de 2 GB, para ello vamos a utilizar discos virtuales de 1 GB. Modifica el fichero Vagrantfile del ejercicio anterior para crear una nueva máquina.

* **Tarea 1:** Crea una raid llamado md5 con los discos que hemos conectado a la máquina.

~~~
lsblk
   NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda      8:0    0 19.8G  0 disk 
   ├─sda1   8:1    0 18.8G  0 part /
   ├─sda2   8:2    0    1K  0 part 
   └─sda5   8:5    0 1021M  0 part [SWAP]
   sdb      8:16   0    1G  0 disk 
   sdc      8:32   0    1G  0 disk 
   sdd      8:48   0    1G  0 disk 
   sde      8:64   0    1G  0 disk 
   Instalamos el paquete mdadm :
~~~

Instalamos el paquete:
~~~
sudo apt install mdadm
~~~

Creamos el raid 5 de los discos creados:
~~~
sudo mdadm --create /dev/md5 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
   mdadm: Defaulting to version 1.2 metadata
   mdadm: array /dev/md5 started.
~~~

* **Tarea 2:** Comprueba las características del RAID. Comprueba el estado del RAID. ¿Qué capacidad tiene el RAID que hemos creado?

Podemos ver el estado y las características del raid a continuación. Como se muestra el raid 5 tiene 2 GB de capacidad.
~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 3
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:31:24 2019
                State : clean 
       Active Devices : 3
      Working Devices : 3
       Failed Devices : 0
        Spare Devices : 0
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 19
   
       Number   Major   Minor   RaidDevice State
          0       8       16        0      active sync   /dev/sdb
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
~~~

* **Tarea 3:** Crea un volumen lógico de 500Mb en el raid 5.

Instalamos el paquete lvm2 para utilizar volumen lógicos.
~~~
sudo apt install lvm2
~~~

Creamos el volumen físico.
~~~
sudo pvcreate /dev/md5
   Physical volume "/dev/md5" successfully created.
~~~

Creamos el grupo de volúmenes.
~~~
sudo vgcreate raid5vol /dev/md5
   Volume group "raid5vol" successfully created
~~~

Creamos el volumen logico de 500Mb.
~~~
sudo lvcreate raid5vol -L 500M -n storage
   Logical volume "storage" created.
~~~

Comprobamos:
~~~
lsblk
   NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   sda                    8:0    0 19.8G  0 disk  
   ├─sda1                 8:1    0 18.8G  0 part  /
   ├─sda2                 8:2    0    1K  0 part  
   └─sda5                 8:5    0 1021M  0 part  [SWAP]
   sdb                    8:16   0    1G  0 disk  
   └─md5                  9:5    0    2G  0 raid5 
     └─raid5vol-storage 253:0    0  500M  0 lvm   
   sdc                    8:32   0    1G  0 disk  
   └─md5                  9:5    0    2G  0 raid5 
     └─raid5vol-storage 253:0    0  500M  0 lvm   
   sdd                    8:48   0    1G  0 disk  
   └─md5                  9:5    0    2G  0 raid5 
     └─raid5vol-storage 253:0    0  500M  0 lvm   
   sde                    8:64   0    1G  0 disk  
~~~

* **Tarea 4:** Formatea ese volumen con un sistema de archivo xfs.

Instalamos el paquete xfsprogs para poder formatear en xfs.
~~~
sudo apt install xfsprogs
~~~

Formateamos el volumen lógico raid5vol
~~~
sudo mkfs.xfs -f /dev/mapper/raid5vol-storage 
   meta-data=/dev/mapper/raid5vol-storage isize=512    agcount=8, agsize=16000 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=1, sparse=1, rmapbt=0
            =                       reflink=0
   data     =                       bsize=4096   blocks=128000, imaxpct=25
            =                       sunit=128    swidth=256 blks
   naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
   log      =internal log           bsize=4096   blocks=896, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
~~~

Comprobamos:
~~~
lsblk -f
   NAME            FSTYPE            LABEL  UUID                                      FSAVAIL FSUSE% MOUNTPOINT
   sda                                                                                               
   ├─sda1          ext4                     b9ffc3d1-86b2-4a2c-a8be-f2b2f4aa4cb5        16.4G     6% /
   ├─sda2                                                                                            
   └─sda5          swap                     f8f6d279-1b63-4310-a668-cb468c9091d8                     [SWAP]
   sdb             linux_raid_member Raid:5 cbbc1438-60a2-8acf-88f3-a2aff362407e                     
   └─md5           LVM2_member              jcDLz9-CZ0w-wz61-FtH4-JK6z-WOjo-RhyqBL                   
     └─raid5vol-storage
                   xfs                      6506a6de-820a-4f0c-befc-40a4422f53cd                     
   sdc             linux_raid_member Raid:5 cbbc1438-60a2-8acf-88f3-a2aff362407e                     
   └─md5           LVM2_member              jcDLz9-CZ0w-wz61-FtH4-JK6z-WOjo-RhyqBL                   
     └─raid5vol-storage
                   xfs                      6506a6de-820a-4f0c-befc-40a4422f53cd                     
   sdd             linux_raid_member Raid:5 cbbc1438-60a2-8acf-88f3-a2aff362407e                     
   └─md5           LVM2_member              jcDLz9-CZ0w-wz61-FtH4-JK6z-WOjo-RhyqBL                   
     └─raid5vol-storage
                   xfs                      6506a6de-820a-4f0c-befc-40a4422f53cd                     
   sde          
~~~

* **Tarea 5:** Monta el volumen en el directorio /mnt/raid5 y crea un fichero. ¿Qué tendríamos que hacer para que este punto de montaje sea permanente?

Creamos el directorio dentro de mnt raid5, montamos el volumen raid5vol dentro y creamos un fichero.
~~~
sudo mkdir /mnt/raid5
~~~

~~~
sudo mount /dev/mapper/raid5vol-storage /mnt/raid5/
~~~

~~~
sudo touch /mnt/raid5/pruebaraid5.txt
~~~

~~~
ls /mnt/raid5/
   pruebaraid5.txt
~~~

Para que el montaje se mantenga tenemos que configurar el fichero /etc/fstab

* **Tarea 6:** Marca un disco como estropeado. Muestra el estado del raid para comprobar que un disco falla. ¿Podemos acceder al fichero?

Marcamos el disco /dev/sdb como estropeado y comprobamos que podemos acceder al fichero
~~~
sudo mdadm /dev/md5 -f /dev/sdb
   mdadm: set /dev/sdb faulty in /dev/md5
~~~

~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 3
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:39:42 2019
                State : clean, degraded 
       Active Devices : 2
      Working Devices : 2
       Failed Devices : 1
        Spare Devices : 0
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 25
   
       Number   Major   Minor   RaidDevice State
          -       0        0        0      removed
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
   
          0       8       16        -      faulty   /dev/sdb
~~~

~~~
ls /mnt/raid5/
   pruebaraid5.txt
~~~

* **Tarea 7:** Una vez marcado como estropeado, lo tenemos que retirar del raid.

Quitamos el disco dañado /dev/sdb del raid 5.
~~~
sudo mdadm /dev/md5 --remove /dev/sdb
   mdadm: hot removed /dev/sdb from /dev/md5
~~~

~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 2
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:42:31 2019
                State : clean, degraded 
       Active Devices : 2
      Working Devices : 2
       Failed Devices : 0
        Spare Devices : 0
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 28
   
       Number   Major   Minor   RaidDevice State
          -       0        0        0      removed
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
~~~

* **Tarea 8:** Imaginemos que lo cambiamos por un nuevo disco nuevo (el dispositivo de bloque se llama igual), añádelo al array y comprueba como se sincroniza con el anterior.

Añadimo de nuevo el disco /dev/sdb
~~~
sudo mdadm /dev/md5 --add /dev/sdb
   mdadm: added /dev/sdb
~~~

Comprobamos como se esta sincronizando, se muestra con el estado de spare rebuilding
~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 3
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:46:19 2019
                State : clean, degraded, recovering 
       Active Devices : 2
      Working Devices : 3
       Failed Devices : 0
        Spare Devices : 1
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
       Rebuild Status : 31% complete
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 35
   
       Number   Major   Minor   RaidDevice State
          4       8       16        0      spare rebuilding   /dev/sdb
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
~~~

A los pocos segundos mostramos el estado del raid 5 y comprobamos que ya se ha sincronizado.
~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 3
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:46:23 2019
                State : clean 
       Active Devices : 3
      Working Devices : 3
       Failed Devices : 0
        Spare Devices : 0
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 47
   
       Number   Major   Minor   RaidDevice State
          4       8       16        0      active sync   /dev/sdb
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
~~~

* **Tarea 9:** Añade otro disco como reserva. Vuelve a simular el fallo de un disco y comprueba como automática se realiza la sincronización con el disco de reserva.

Añadimos de forma normal el nuevo disco /dev/sde pero al estar configurado el raid 5 con 3 dispositivos, este se pondra en estado de spare
~~~
sudo mdadm /dev/md5 --add /dev/sde
   mdadm: added /dev/sde
~~~

~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 4
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:55:38 2019
                State : clean 
       Active Devices : 3
      Working Devices : 4
       Failed Devices : 0
        Spare Devices : 1
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 48
   
       Number   Major   Minor   RaidDevice State
          4       8       16        0      active sync   /dev/sdb
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
   
          5       8       64        -      spare   /dev/sde
~~~

Ahora realizamos el fallo del disco /dev/sdb y comprobamos como se sincroniza y luego como esta sincronizado.
~~~
sudo mdadm /dev/md5 -f /dev/sdb
   mdadm: set /dev/sdb faulty in /dev/md5
~~~

~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 4
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:56:12 2019
                State : clean, degraded, recovering 
       Active Devices : 2
      Working Devices : 3
       Failed Devices : 1
        Spare Devices : 1
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
       Rebuild Status : 19% complete
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 53
   
       Number   Major   Minor   RaidDevice State
          5       8       64        0      spare rebuilding   /dev/sde
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
   
          4       8       16        -      faulty   /dev/sdb
~~~

~~~
sudo mdadm --detail /dev/md5 
   /dev/md5:
              Version : 1.2
        Creation Time : Sat Oct  5 10:31:12 2019
           Raid Level : raid5
           Array Size : 2093056 (2044.00 MiB 2143.29 MB)
        Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
         Raid Devices : 3
        Total Devices : 4
          Persistence : Superblock is persistent
   
          Update Time : Sat Oct  5 10:56:16 2019
                State : clean 
       Active Devices : 3
      Working Devices : 3
       Failed Devices : 1
        Spare Devices : 0
   
               Layout : left-symmetric
           Chunk Size : 512K
   
   Consistency Policy : resync
   
                 Name : Raid:5  (local to host Raid)
                 UUID : cbbc1438:60a28acf:88f3a2af:f362407e
               Events : 67
   
       Number   Major   Minor   RaidDevice State
          5       8       64        0      active sync   /dev/sde
          1       8       32        1      active sync   /dev/sdc
          3       8       48        2      active sync   /dev/sdd
   
          4       8       16        -      faulty   /dev/sdb
~~~

* **Tarea 10:** Redimensiona el volumen y el sistema de archivo de 500Mb al tamaño del raid.

Miramos cuanto espacio libre queda en el grupo de volumenes:
~~~
sudo vgs
   VG       #PV #LV #SN Attr   VSize VFree
   raid5vol   1   1   0 wz--n- 1.99g 1.50g
~~~

Utilizamos lvextend para redimensionar el volumen 1.5Gb extras y ponemos la opción -r para que redimensione también el sistema de fichero.
~~~
sudo lvextend /dev/mapper/raid5vol-storage -L +1.5G -r
     Size of logical volume raid5vol/storage changed from 500.00 MiB (125 extents) to    <1.99 GiB (509 extents).
     Logical volume raid5vol/storage successfully resized.
   meta-data=/dev/mapper/raid5vol-storage isize=512    agcount=8, agsize=16000 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=1, sparse=1, rmapbt=0
            =                       reflink=0
   data     =                       bsize=4096   blocks=128000, imaxpct=25
            =                       sunit=128    swidth=256 blks
   naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
   log      =internal log           bsize=4096   blocks=896, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
   realtime =none                   extsz=4096   blocks=0, rtextents=0
   data blocks changed from 128000 to 521216
~~~

~~~
df -h
   Filesystem                    Size  Used Avail Use% Mounted on
   udev                          228M     0  228M   0% /dev
   tmpfs                          49M  3.6M   45M   8% /run
   /dev/sda1                      19G  1.1G   17G   7% /
   tmpfs                         242M     0  242M   0% /dev/shm
   tmpfs                         5.0M     0  5.0M   0% /run/lock
   tmpfs                         242M     0  242M   0% /sys/fs/cgroup
   tmpfs                          49M     0   49M   0% /run/user/1000
   /dev/mapper/raid5vol-storage  2.0G   29M  2.0G   2% /mnt/raid5
~~~