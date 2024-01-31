---
layout: post
title: "Finding ZFS Snapshots Containing a File or Directory"
date: 2024-01-31 22:55:42
categories: posts
---

A great feature of the ZFS filesystem is its ability to create snapshots of datasets.
When using snapshots as part of an incremental backup strategy, such as through [zfs-auto-snapshots](https://github.com/zfsonlinux/zfs-auto-snapshot), you may encounter situations where these snapshots consume a considerable amount of space.

This scenario often arises when working with large media or image files.
If such files are moved from one location that is being snaoshotted to another, they might inadvertently become part of a snapshot.
Ideally, such operations should be done on a separate, non-snapshotted dataset or volume, but mistakes are easily made.

Since snapshots are read-only, the only way of freeing up the occupied space is by removing all snapshots that contain the respective file or directory.
While the convenience of not having to worry about file deletion is appreciated, it comes with the drawback that you cannot remove them even if you are sure that they are no longer needed.

If you promptly notice a file or directory being accidentally included in a snapshot, deleting the most recent snapshot suffices.
However, if time has passed before recognizing the issue, you need to remove all snapshots that include the files or directories.

Manually locating and removing the snapshots containing the relevant files or directories can be rather time-consuming.
First, You need to search for a file or directory within the `.zfs/snapshot` directory and then reconstruct the snapshot name by prepending the dataset name.

Having encountered this issue multiple times, I created a script to automate the process: [zfs-find-snapshots](https://github.com/kevinmorio/zfs-find-snapshots)
This script lists all snapshots containing a specified file or directory.

For instance, let's assume you have a dataset `tank` with three snapshots.

``` shell
$ zfs list -t snapshot tank
NAME     USED  AVAIL  REFER  MOUNTPOINT
test@A    22K      -    24K  -
test@B     0B      -  1.00G  -
test@C     0B      -  1.00G  -
```

You can find all snapshots containing `fileA`.

``` shell
$ zfs-find-snapshot fileA
tank@B
tank@C
```

To remove these snapshots, you can use `xargs`.

``` shell
$ zfs-find-snapshot fileA | xargs -I {} zfs destroy
```

Note that this action may also remove other files you want to preserve.
Make sure you have backups in place before deleting any snapshots.

## Space Usage of Snapshots

You can obtain the space used by snapshots using the `usedbysnapshots` option of the `zfs list` command.

``` shell
$ zfs list -o name,used,usedbysnapshots tank
NAME   USED  USEDSNAP
tank  1.00G     1.00G
```

The output shows that the `tank` dataset uses one gigabyte of space in total, entirely allocated to the snapshots.

By specifying the type as `snapshot`, you can get a list showing the space used by each individual snapshot.

``` shell
$ zfs list -t snapshot blek
NAME     USED  AVAIL  REFER  MOUNTPOINT
tank@A    22K      -    24K  -
tank@B  1.00G      -  1.00G  -
tank@C     0B      -    24K  -
```

In this example, `tank@A` was created on an empty directory, `tank@B` on a single one-gigabyte file, which was subsequently removed before `tank@C` was created.
As expected, the one-gigabyte file is stored within the `tank@B` snapshot.

## References

- [Disk Space Accounting for ZFS Snapshots](https://docs.oracle.com/cd/E19253-01/819-5461/gbcxc/index.html).
- [ZFS Native Property Descriptions](https://docs.oracle.com/cd/E19253-01/819-5461/6n7ht6r2s/index.html#gcfgr)
- [zfs-auto-snapshots](https://github.com/zfsonlinux/zfs-auto-snapshot)
- [zfs-find-snapshots](https://github.com/kevinmorio/zfs-find-snapshots)
