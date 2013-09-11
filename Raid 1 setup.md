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

