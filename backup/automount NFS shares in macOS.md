# The Read-only Root Filesystem

```
synthetic.conf – synthetic symbolic link and directory manifest
```
This file allows creating symbol links/empty directoreis under / which is mounted as read-only.

# /etc/auto_master

```
#
# Automounter master map
#
+auto_master  # Use directory service
#/net   -hosts  -nobrowse,hidefromfinder,nosuid
/home   auto_home -nobrowse,hidefromfinder
/Network/Servers -fstab
/-   -static
/-   auto_nfs -soft,nobrowse,nosuid
```

# /etc/auto_nfs

```
/System/Volumes/Data/DVD -fstype=nfs,soft,resvport,rw,noowners 192.168.1.10:/DVD
/System/Volumes/Data/Bluray -fstype=nfs,soft,resvport,rw,noowners 192.168.1.10:/Bluray
```

# /etc/synthetic.conf

```
DVD^ISystem/Volumes/Data/DVD
Bluray^ISystem/Volumes/Data/Bluray
```

# /etc/nfs.conf for the NFS server

```
nfs.client.mount.options = vers=4
```