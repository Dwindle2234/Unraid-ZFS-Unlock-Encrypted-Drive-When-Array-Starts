# Unraid-ZFS-Unlock-Encrypted-Drive-When-Array-Starts
Instructions and simple script that will unlock your encrypted ZFS shares by pulling a keyfile via SSH from another device. No human interaction needed

# ZFS Automatically Unlocking Encrypted Shares
Principle
---------

Principle is that the password file (the key to unlocking datasets) sits (in my case) on a local pfSense/Netgate device.  Any local SSH device would be suitable

When unRaid starts up the password file is pulled onto the transiant /root folder by a script

If the file exists then the drive is mounted

If unRaid server is stolen then there is should be no access to Netgate and hence drives remain encrypted

Script Setup
-----

To get it working install plugin NERDTOOLS and enable SSHPASS. This enables us to SSH into NETGATE within a script without user interaction.

On NETGATE we use add-on **FILER** to create a file in a users (SSH\_USER) home folder.  This user only has access for SSHing (effectively ADMIN rights - no advised to expose this publically)

The script runs on unraid thanks to plugin USER\_SCRIPTS. Set the script to run when 'array starts up'

Creating New Encrypted DataSets
---------------------------

```text-x-sh
zfs create -o encryption=on -o keylocation=file:///root/unraidpass.txt -o keyformat=passphrase disk1/SHARE-NAME
```

Be sure to create on as many disks as required - and **set the share to only use those disk**

This command lists the result

```text-x-sh
zfs get keylocation disk1/SHARE-NAME
```

You can 'lock' then 'unnlock' using the GUI (ZFS MASTER plugin) without needed the password (provided you haven't deleted it from /root)

Copying Files Over from existing unencrypted share
------------------

Suggest existing share is renamed, new dataset created then rsyc the files over

```text-x-sh
rsync -av --stats --progress /mnt/user/general/ /mnt/user/general
```
NOTE the extra slash on the source!


Changing an Existing Encrypted Share to Use the Password File
-------------------------------------------------------------

```text-x-sh
zfs change-key -o keylocation=file:///root/unraidpass.txt -o keyformat=passphrase disk1/SHARE-NAME
```

This doesn't change the 'master' ZFS password so the drive won't be recrypted with this command


Reference
---------

This is very useful: [https://arstechnica.com/gadgets/2021/06/a-quick-start-guide-to-openzfs-native-encryption/](https://arstechnica.com/gadgets/2021/06/a-quick-start-guide-to-openzfs-native-encryption/)


Credit
------
Thanks to SpaceInvaderOne for inspiring the idea (he used a FTP site to unlock the entire array): https://youtu.be/TSlHEBR1yfY?si=ixdkU-hyi6_eYLZk
