# ZFS Unlocking Encrypted Shares without Human Input - and ensuring it only works on your own network
Instructions and simple script that will unlock your encrypted ZFS shares by pulling a keyfile via SSH from another device. No human interaction needed.  Drive contents should be unreadable by whover nicked the server

Principle
---------

Principle is that the password file (the key to unlocking datasets) sits (in my case) on a local pfSense/Netgate device.  Any local SSH device would be suitable. My netgate is well hidden!

When unRaid starts up the password file is pulled onto the transiant /root folder by a script

If the file exists then the drive(s) are mounted

If unRaid server is stolen then there should be no access to Netgate and hence drives remain encrypted

Setup
-----

To enable the script to run on UnRaid install plugin ```NERDTOOLS``` and enable ```SSHPASS```. This enables us to SSH into NETGATE within a script without user interaction.

Edit in this [script](UnraidScript.txt) using the User\_Scripts plugin. Set the script to run when 'array starts up'

On NETGATE device use add-on ```FILER``` to create a file in a user's (SSH\_USER) home folder.  The file should be plain text and contain a password - name the file ```unraidpass.txt```.

Create a new pfSense user and grant access for SSHing only (effectively gives ADMIN rights - sop not advised to expose port 22 publically!).

Also enable SSH on the pfSense - Sytem/Advanced/Secure Shell. Leave the other options as default.


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

Suggest existing share is renamed (use unRaid GUI - SHARES tab) and a new dataset created in its place - then rsyc the files over

```text-x-sh
rsync -av --stats --progress /mnt/user/general-17-aug/ /mnt/user/general
```
NOTE the extra slash at the end of the source path!


Changing an Existing Encrypted Share to Use the Password File
-------------------------------------------------------------

```text-x-sh
zfs change-key -o keylocation=file:///root/unraidpass.txt -o keyformat=passphrase disk1/SHARE-NAME
```

This doesn't change the 'master' ZFS password so the drive won't be recrypted with this command.  The change is instant.

Procedure should server be nicked
---

Nothing to do really....you should maybe change the SSH_USER's password on pfSense.  But unless they can plug back into your network then there's no risk. 


Reference
---------

This is very useful: [https://arstechnica.com/gadgets/2021/06/a-quick-start-guide-to-openzfs-native-encryption/](https://arstechnica.com/gadgets/2021/06/a-quick-start-guide-to-openzfs-native-encryption/)


Credits
------
Thanks to [SpaceInvaderOne](https://youtu.be/TSlHEBR1yfY?si=ixdkU-hyi6_eYLZk) for inspiring the idea (he used a FTP site to unlock the entire array)
