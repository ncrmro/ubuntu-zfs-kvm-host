## Readme Layout
* [Ansible](#ansible) - software that automates software provisioning, configuration management, and application deployment
* [KVM](#kvm) - virtualization infrastructure for the Linux kernel that turns it into a hypervisor
* [ZFS](#zfs) - file system and logical volume manager
 * [Inspecting ZFS](#inspecting-zfs)
 * [Snapshoting](#snapshotting)
 * [Adding Hardware](#adding-hardware)
* [Provisioning the host](#provisioning-the-linux-host) - Incase of catastropic failure steps to rebuild root kvm host.
* [Stack Choices](#stack-choices) - Reasons and benifits of each tech used in stack.


## Getting Started

When working in the development environment you will need vagrant to create simulated kvm-host and python for ansible.

Install hombrew-cask, vagrant and it's dependencies
`brew tap caskroom/cask`

`brew cask install virtualbox vagrant vagrant-manager`

Install python3 and pipenv
`brew install python3 pipenv`

Upgrade pip and install virtualenv
`pip3 install --upgrade pip virtualenv`

Create the virtualenv
`virtualenv ~/.local/share/virtualenvs/kvm-host`

Activate the virtualenv
`source ~/.venv/kvm-host/bin/activate`

Install project dependencies
`pipenv install`

### Ansible

To get started working with ansible in development first run `vagrant up`

SSH into your vagrant box `vagrant ssh`


#### Ansible Vault
Create a password file

```
mkdir ~/.ansible ; \
echo "mypassword" >> ~/.ansible/.vault_pass.txt && \
chmod 0444 ~/.ansible/.vault_pass.txt
```

To generate the secret file, run `ansible-vault create vars/secret` from the project root.

The first prompt will create your vault password.

The second prompt will send you to a vim session, in that vim sesson input:
 
`ansible_sudo_pass: your-sudo-password`

## KVM
Widly supported across linux as the defacto virtualization infrastructure even AWS has transistioned from xen to KVM.

Inspecting our VM's
`virsh list --all`

This would give us a list of all vms.
```
 Id    Name                           State
----------------------------------------------------
 1     dnode0                         running
 2     bastion                        running
 -     ubuntu18_04_server             shut off
```

We can reboot machines with reboot `virsh reboot dnode0`

## ZFS

The operating system and storage drives use ZFS. Thus we gan all the benifits of ZFS on our root drive and storage.


### Inspecting ZFS

To see our root zfs pool "rpool" for "root pool"
`zpool list`

```
NAME    SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
rpool   236G  4.47G   232G         -     0%     1%  1.00x  ONLINE  -
```


list all devices and mount points
`sudo lsblk -o NAME,FSTYPE,SIZE,TYPE,MOUNTPOINT,LABEL`

```
NAME   FSTYPE       SIZE MOUNTPOINT LABEL
sda    zfs_member 238.5G            rpool
├─sda1 zfs_member   238G            rpool
└─sda3 vfat         512M /boot/efi  EFI
sdb               931.5G            
├─sdb1 zfs_member 931.5G            hdd1
└─sdb9                8M   
```
Here we can confirm that device sda is a zfs member with rpool as the label.

Using the zfs command we can inspect rpool
`zfs list -r rpool`

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
rpool/var/log      7.16M   224G  7.16M  legacy
```

### Snapshotting

List all the snapshots with `zfs list -t snapshot`

```
NAME                        USED  AVAIL  REFER  MOUNTPOINT
rpool/ROOT/ubuntu@install  13.4M      -   818M  -
```

We can see the snapshot made at install.

#### New snapshot

Lets go ahead and make a new snapshots `zfs snapshot rpool/ROOT/ubuntu@test123`.
```
NAME                        USED  AVAIL  REFER  MOUNTPOINT
rpool/ROOT/ubuntu@install  13.4M      -   818M  -
rpool/ROOT/ubuntu@test123     0B      -  4.30G  -
```

We can now see the new snapshot we made

### Turning hardware into zfs pools


### Adding a new storage device
List all the storage drives
`ls /dev/disk/by-id/`

```
ata-M4-CT256M4SSD2_00000000123509146D38        ata-ST1000DM003-1SB102_Z9A59DHH-part1
ata-M4-CT256M4SSD2_00000000123509146D38-part1  ata-ST1000DM003-1SB102_Z9A59DHH-part9
ata-M4-CT256M4SSD2_00000000123509146D38-part3
ata-ST1000DM003-1SB102_Z9A59DHH
```

In this case the device we want is `/dev/disk/by-id/ata-ST1000DM003-1SB102_Z9A59DHH`

While our root device is `/dev/disk/by-id/ata-M4-CT256M4SSD2_00000000123509146D38`

Lets clean the disk with `sgdisk --zap-all /dev/disk/by-id/ata-ST1000DM003-1SB102_Z9A59DHH`

#### Creating the new pool

Now that we have the device we can choose some options such as where to mount and weather or not to enable compression.

```
zpool create \
      -O compression=lz4 \
      hdd1 /dev/disk/by-id/ata-ST1000DM003-1SB102_Z9A59DHH
```

Finally leats create a new zfs on the hdd1 pool and mount it at hdd1/timemachine 

`zfs create -o mountpoint=/mnt/timemachine hdd1/timemachine`

### Further reading

* [ZFS to Dedupe or not to Dedupe](https://constantin.glez.de/2011/07/27/zfs-to-dedupe-or-not-dedupe/)
* [Ansible Playbooks best practices: Alternative Directory Layout](http://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#alternative-directory-layout)

## Provisioning the linux host.

Create and ubuntu 18.04 LTS Desktop USB disk.

Follow guide [here](https://github.com/zfsonlinux/zfs/wiki/Ubuntu-18.04-Root-on-ZFS)

Condensed version of that guides commands can be found [here](https://gist.github.com/ncrmro/f389c1362baf4d19d6e8b310d66902e6#file-zfsubuntubionic-md)

## Stack Choices
* Ubuntu
* KVM - Widly supported across linux as the defacto virtualization infrastructure even AWS has transistioned from xen to KVM.
* ZFS
 * [Data integrity](https://en.wikipedia.org/wiki/ZFS#Data_integrity)
 * [Native raid with zfs](https://en.wikipedia.org/wiki/ZFS#RAID_(%22RaidZ%22))
 * [Avoidance of hardware RAID controllers](https://en.wikipedia.org/wiki/ZFS#Avoidance_of_hardware_RAID_controllers)
 * compression: data is compressed
 * deduplication: only one copy of duplicate data is stored
 * [Snapshots and clones](https://en.wikipedia.org/wiki/ZFS#Snapshots_and_clones)
        * [Sending and receving snapshots to remote hosts](https://en.wikipedia.org/wiki/ZFS#Sending_and_receiving_snapshots)
