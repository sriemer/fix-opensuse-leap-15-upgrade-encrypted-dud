# openSUSE Leap 15 Upgrade Fix

## Problem Description

If you want to upgrade from Leap 42.3 to Leap 15 with an encrypted home
partition, then you run into the bug
https://bugzilla.suse.com/show_bug.cgi?id=1094963

It throws an exception right when selecting the distro to be upgraded:
```
Details: Storage::Exception
Caller:/mounts/mp_0001/usr/share/YaST2/lib/y2storage/storage_class_wrapper.rb:260: in `find_by_any_name`
```

## Building the Fix

Only updated YaST2 and `libstorage-ng` files fix this. So I have prepared a
Driver Update Disk (DUD) `y2lp15.dud` here with files which were latest on
**2018-11-29**.

**File source**: http://download.opensuse.org/update/leap/15.0/oss/x86_64/

**File list:**
```
libstorage-ng1-3.3.315-lp150.2.9.1.x86_64.rpm
libstorage-ng-python3-3.3.315-lp150.2.9.1.x86_64.rpm
libstorage-ng-ruby-3.3.315-lp150.2.9.1.x86_64.rpm
libstorage-ng-utils-3.3.315-lp150.2.9.1.x86_64.rpm
yast2-4.0.87-lp150.2.9.1.x86_64.rpm
yast2-bootloader-4.0.39-lp150.2.8.1.x86_64.rpm
yast2-core-4.0.4-lp150.2.6.1.x86_64.rpm
yast2-storage-ng-4.0.214-lp150.2.15.1.x86_64.rpm
yast2-update-4.0.18-lp150.2.6.1.x86_64.rpm
```

Command used to build the DUD:
```
mkdud --create y2lp15.dud --dist leap15.0 --name "Upgrade fix boo#1094963" for-lp15-dud/*
```

Command to show its contents:
```
mkdud --show y2lp15.dud
```

## Applying the DUD

Select the upgrade and use the following upgrade boot options:
```
dud=ftp://your_ftpserver/y2lp15.dud insecure=1
```

If you want to know what the installation initrd `linuxrc` is doing and if it
applies the DUD properly, then press **Esc** when the green bar is visible at the
bottom pretending that some progress is going on. You can also add the `startshell=1`
boot option to get more control.

I have tested this method. Using this DUD fixes this bug for me.

## Applying the DUD from USB

Besides network install, it is also possible to extract DUDs (which are `cpio.gz`
archives) to the root of a filesystem which supports symlinks like e.g. `ext3`.
Let us assume I have a USB stick with only one ext3 partition mounted to `/mnt`.

Then I use the following commands:
```
cp y2lp15.dud /mnt; cd /mnt
zcat y2lp15.dud | cpio -idmv
cd ~; sudo umount /mnt
```

A directory `linux` should appear. `linuxrc` is looking for that directory on
all partitions and picks up the DUD automatically this way.
