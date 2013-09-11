###Setting up RAID 1 using mdadm

Backing up data isn't as difficult at it appears.  mdadm is a great tool to acoomodate all types of RAID setups.  This write up is to document the steps I went through in order to set up a RAID 1 device, verify it works, kill a drive, and how to rebuild an array with a failed drive.

####*mdadm*
It's a Linux utility for managing software raid devices.  It's name is derrived from **md** (multiple device) device nodes it **adm**inisters or manages.  It's capable of creating:

* RAID 0
* RAID 1
* RAID 4
* RAID 5
* RAID 6
* RAID 10

####RAID 1
This setup is used to create exact copies (mirrors) of the data on two disks.

**pro:** Redundancy is increased by the power of the number of devices

**con:** Storage capacity.  The array can only be as big as the smallest member disk.

####Marking a drive as faulty

rrivas@gerald ~ $ sudo mdadm --detail /dev/md1

    /dev/md1:
            Version : 1.2
      Creation Time : Fri Jun 21 13:01:36 2013
         Raid Level : raid1
         Array Size : 2930133824 (2794.39 GiB 3000.46 GB)
      Used Dev Size : 2930133824 (2794.39 GiB 3000.46 GB)
       Raid Devices : 2
      Total Devices : 2
        Persistence : Superblock is persistent
    
        Update Time : Tue Sep 10 20:04:34 2013
              State : clean
     Active Devices : 2
    Working Devices : 2
     Failed Devices : 0
      Spare Devices : 0

               Name : gerald:1  (local to host gerald)
               UUID : 5a4cdaea:390c699b:7bf07b05:10a86c77
             Events : 41

        Number   Major   Minor   RaidDevice State
           0       8       49        0      active sync   /dev/sdd1
           1       8       65        1      active sync   /dev/sde1
           
rrivas@gerald ~ $ sudo mdadm /dev/md1 -f /dev/sdd1

*mdadm: set /dev/sdd1 faulty in /dev/md1*

rrivas@gerald ~ $ sudo mdadm --detail /dev/md1

    /dev/md1:
            Version : 1.2
      Creation Time : Fri Jun 21 13:01:36 2013
         Raid Level : raid1
         Array Size : 2930133824 (2794.39 GiB 3000.46 GB)
      Used Dev Size : 2930133824 (2794.39 GiB 3000.46 GB)
       Raid Devices : 2
      Total Devices : 2
        Persistence : Superblock is persistent
    
        Update Time : Tue Sep 10 20:18:52 2013
              State : clean, degraded
     Active Devices : 1
    Working Devices : 1
     Failed Devices : 1
      Spare Devices : 0

               Name : gerald:1  (local to host gerald)
               UUID : 5a4cdaea:390c699b:7bf07b05:10a86c77
             Events : 43

        Number   Major   Minor   RaidDevice State
           0       0        0        0      removed
           1       8       65        1      active sync   /dev/sde1

           0       8       49        -      faulty spare   /dev/sdd1
           
rrivas@gerald ~ $ sudo mdadm /dev/md1 -r /dev/sdd1

*mdadm: hot removed /dev/sdd1 from /dev/md1*

    /dev/md1:
            Version : 1.2
      Creation Time : Fri Jun 21 13:01:36 2013
         Raid Level : raid1
         Array Size : 2930133824 (2794.39 GiB 3000.46 GB)
      Used Dev Size : 2930133824 (2794.39 GiB 3000.46 GB)
       Raid Devices : 2
      Total Devices : 1
        Persistence : Superblock is persistent
    
        Update Time : Tue Sep 10 20:23:55 2013
              State : clean, degraded
     Active Devices : 1
    Working Devices : 1
     Failed Devices : 0
      Spare Devices : 0
    
               Name : gerald:1  (local to host gerald)
               UUID : 5a4cdaea:390c699b:7bf07b05:10a86c77
             Events : 46

        Number   Major   Minor   RaidDevice State
           0       0        0        0      removed
           1       8       65        1      active sync   /dev/sde1


####Replacing a faulty member of the RAID

Remove the hard drive that was just failed.  You should label this drive to the corresponding */dev/sd\** so it's easier to figure which drive to pull out.

I have inserted a new hard drive and it mounted as `/dev/sdd`

rrivas@gerald ~ $ sudo mdadm /dev/md1 -a /dev/sdd
mdadm: added /dev/sdd


rrivas@gerald ~ $ sudo mdadm --detail /dev/md1

    /dev/md1:
            Version : 1.2
      Creation Time : Fri Jun 21 13:01:36 2013
         Raid Level : raid1
         Array Size : 2930133824 (2794.39 GiB 3000.46 GB)
      Used Dev Size : 2930133824 (2794.39 GiB 3000.46 GB)
       Raid Devices : 2
      Total Devices : 2
        Persistence : Superblock is persistent

        Update Time : Tue Sep 10 22:11:59 2013
              State : clean, degraded, recovering
     Active Devices : 1
    Working Devices : 2
     Failed Devices : 0
      Spare Devices : 1
    
     Rebuild Status : 0% complete

               Name : gerald:1  (local to host gerald)
               UUID : 5a4cdaea:390c699b:7bf07b05:10a86c77
             Events : 66

        Number   Major   Minor   RaidDevice State
           2       8       48        0      spare rebuilding   /dev/sdd
           1       8       65        1      active sync   /dev/sde1



rrivas@gerald ~ $ cat /proc/mdstat

    Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
    md1 : active raid1 sde1[1] sdd[2]
          2930133824 blocks super 1.2 [2/1] [_U]
          [>....................]  recovery =  4.0% (118921216/2930133824)         finish=235.2min speed=199182K/sec






failing a drive: http://bencane.com/2011/07/05/mdadm-manually-fail-a-drive/
