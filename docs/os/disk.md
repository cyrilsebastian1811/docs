# __:material-harddisk:{.lg .top} Disks__


In Linux, a filesystem is a hierarchy of directories (also referred to as a directory tree) that is used to organize files on your computer. The filesystem is always rooted at `/`.


``` { .bash .no-copy }
df -h

# Output
/dev/sda4                                       37G   37G   20K 100% /       # (1)!
tmpfs                                           48G   40K   48G   1% /tmp    # (2)!
/dev/sdb1                                      950G  9.4G  941G   1% /app    # (3)!
```

1.  `/dev/sda4 37G 37G 20K 100% /`: This line indicates that the filesystem on the device `/dev/sda4` is mounted at the root directory (`/`). It's almost full, using 100% of its 37G capacity.

2.  `tmpfs 48G 40K 48G 1% /tmp`: This line indicates that a tmpfs filesystem is mounted at `/tmp`. tmpfs is a temporary filesystem that stores files in memory, not on a disk. It's using very little of its 48G capacity.

3.  `/dev/sdb1 950G 9.4G 941G 1% /app`: This line indicates that the filesystem on the device `/dev/sdb1` is mounted at `/app`. It's using very little of its 950G capacity.

Even though `/app` is a subdirectory of `/`, it's mounted on a different filesystem. This means that when you navigate to `/app` and its subdirectories, you're working with files on the `/dev/sdb1` device, not the `/dev/sda4` device. Any space usage or file changes in `/app` do not affect the `/dev/sda4` filesystem, and vice versa.

This is a common setup for servers where you want to separate the operating system files (on `/`) from application data (on `/app`), especially when the application data needs a lot of space. If `/app` fills up, it won't affect the operating system files on `/`.


In Linux, a file system can be mounted at multiple points in the directory tree. This is done using the `mount --bind` command. Here's an example:

``` { .bash .copy }
mount --bind /original/mount/point /new/mount/point
```

In this command, replace `/original/mount/point` with the original mount point of the file system, and replace `/new/mount/point` with the directory where you want to create the new mount point.

This command makes the file system available at both `/original/mount/point` and `/new/mount/point`. Any changes made in one location will be reflected in the other.

Please note that this is a temporary mount: it will not persist after a reboot. If you want to make it permanent, you should add it to your `/etc/fstab` file.


## Commands


1. `df`: The `df` command stands for "disk filesystem". It is used to display the amount of disk space used and available on Linux and Unix filesystems. By default, `df` displays space in 1K blocks. You can use the `-h` option to make it display space in a "human-readable" format (e.g., KB, MB, GB).

2. `du`: The `du` command stands for "disk usage". It is used to estimate file and directory space usage. The `du` command can be used to track the files and directories which are consuming excessive amount of space on hard disk drive. Like `df`, `du` also has a `-h` option for human-readable output. The command syntax is `du -h --max-depth=1 /path`

3. `lsblk`: The lsblk command lists information about all available or the specified block devices. Block devices are storage devices such as hard drives, flash drives, and CD-ROMs.

4. `fdisk`: The fdisk command is a text-based utility for viewing and managing hard disk partitions on Linux. It's one of the most powerful tools you can use to manage partitions, but it's also very dangerous because it's easy to make mistakes.

5. `mount`: The `mount` command is used to `mount` filesystems. The command syntax is `mount [-l|-h|-V]` or `mount -a [-fFnrsvw] [-t vfstype] [-O optlist]` or `mount [-fnrsvw] [-o options [,...]] device | dir`.

6. `umount`: The `umount` command is used to unmount filesystems. The command syntax is `umount -h` or `umount -V` or `umount [-fkv] [--] dir | device [...]`.

7. `fsck`: The `fsck` command stands for "file system check". It is used to check and optionally repair one or more Linux file systems.

8. `mkfs`: The `mkfs` command stands for "make filesystem". It is used to build a Linux filesystem on a device, usually a hard disk partition.

9. `lsof`: The `lsof` command stands for "list open files". It provides information about files that are opened by processes.

10. `iostat`: The `iostat` command reports Central Processing Unit (CPU) statistics and input/output statistics for devices and partitions. It is a handy tool to measure the load on the input/output devices and filesystems.

11. `hdparm`: The `hdparm` command provides a command line interface to various kernel interfaces supported by the Linux SATA/PATA/SAS "libata" subsystem and the older IDE driver subsystem. It can set parameters such as drive caches, sleep mode, power management, acoustic management, and DMA settings.

12. `smartctl`: The `smartctl` command controls the Self-Monitoring, Analysis and Reporting Technology (SMART) system built into most modern ATA and SCSI harddisks. It is a part of the `smartmontools` package.