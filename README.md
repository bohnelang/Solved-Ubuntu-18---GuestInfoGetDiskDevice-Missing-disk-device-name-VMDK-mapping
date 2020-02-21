# Solved: Linux Ubuntu open-vm-tools: GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/"

After Upgrade from Ubuntu 16 LTS to Ubuntu 18 LTS the open VM-Ware tools write a lot of repeating warnings to the log dir and files like vmware-vmsvc-root.log.
```
[2020-02-21T07:04:23.751Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:04:53.751Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:05:23.754Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:05:53.753Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:06:23.755Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:06:53.755Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:07:23.756Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:07:53.756Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:08:23.759Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:08:53.759Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:09:23.759Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:09:53.759Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:10:23.760Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:10:53.761Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:11:23.761Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:11:53.761Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
[2020-02-21T07:12:23.761Z] [ warning] [guestinfo] GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping unavailable for "/", fsName: "/dev/sda1"
```

**SOLVE and stop this repeating message:**

The **reason is that the VM was configed to an "LSI Logic Parallel SCSI controller"**. This is the problem. 
Better to change the controller to "VMware Paravirtual controller".
1. Shut down VM
2. Change Controller config to "VMware Paravirtual controller"
3. Start VM

Further change/edit the  file /etc/vmware-tools/tools.conf to this

```
[logging]
log = true

vmtoolsd.level = error
vmtoolsd.handler = file
vmtoolsd.data = /var/log/vmtoolsd.${USER}.log

vmsvc.level = error
vmsvc.handler = file
vmsvc.data = /var/log/vmsvc.log

vmusr.level = error
vmusr.handler = file
vmusr.data = /var/log/vmusr.${USER}.log

toolboxcmd.level = error
toolboxcmd.handler = file
toolboxcmd.data = /var/log/vmtoolboxcmd.log
```

(Read: https://kb.vmware.com/s/article/1007873)

And do a
```
 systemctl restart open-vm-tools
```

############################################################


In Ubuntu 19 the bug will be fixed in vmware-tools by changing the priority from Warning to Debug! 
Keep this in mind - the this is **not a warning** - it is a debug info for developer. 

Ubuntu 18 LTS uses 2:11.0.1 but the bug fix will come in in version 2:11.0.5

```
open-vm-tools (2:11.0.5-2) unstable; urgency=medium

  * [eab2f1a] Add vmtoolsd.service alias.
    Debian's open-vm-tools.service is rather unsuaul and based on the
    history of the package, so ship an alias.
  * [b2977cd] Rectify a log spew in vmsvc logging.
    Upstream commit 4ee0bd3c8ead89541ab7d196fb54e940e397420d
    When a LSI Logic Parallel SCSI controller sits in PCI bus 0
    (SCSI controller 0), the Linux disk device enumeration does not provide
    a "label" file with the controller name.  This results in messages like
    "GuestInfoGetDiskDevice: Missing disk device name; VMDK mapping
    unavailable for "/var/log", fsName: "/dev/sda2" repeatedly appearing
    in the vmsvc logging.  The patch converts what previously was a warning
    message to a debug message and thus avoids the log spew.
    Thanks to Oliver Kurth (Closes: #950888)

 -- Bernd Zeimetz <bzed@debian.org>  Tue, 11 Feb 2020 15:56:51 +0100
```
(Read: http://metadata.ftp-master.debian.org/changelogs/main/o/open-vm-tools/unstable_changelog)
