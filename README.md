## Initial setup.

Create and ubuntu 18.04 LTS Desktop USB disk.

Follow guide [here](https://github.com/zfsonlinux/zfs/wiki/Ubuntu-18.04-Root-on-ZFS)

Condensed version of that guides commands can be found [here](https://gist.github.com/ncrmro/f389c1362baf4d19d6e8b310d66902e6#file-zfsubuntubionic-md)

## ZFS

The operating system is installed on ZFS. Thus we gan all the benifits of ZFS on our root drive.

To see our root zfs pool "rpool" for "root pool"
`zpool list`

```
NAME    SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
rpool   236G  4.47G   232G         -     0%     1%  1.00x  ONLINE  -
```

`zfs list`

Thie output should be like so
```
NAME                USED  AVAIL  REFER  MOUNTPOINT
rpool              4.47G   224G    96K  /
rpool/ROOT         4.31G   224G    96K  none
rpool/ROOT/ubuntu  4.31G   224G  4.30G  /
rpool/home         1.16M   224G   104K  /home
rpool/home/username   144K   224G   144K  /home/username
rpool/home/root     940K   224G   940K  /root
rpool/var           163M   224G    96K  /var
rpool/var/cache     156M   224G   156M  /var/cache
rpool/var/log      7.16M   224G  7.16M  legacy
rpool/var/mail       96K   224G    96K  /var/mail
rpool/var/spool     112K   224G   112K  /var/spool
rpool/var/tmp       128K   224G   128K  legacy

```

