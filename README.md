# 1) Description  

kjail is a set of jail management tools. It's based on the sysutils/ezjail principles, but it's simpler, with less options. kjail manages only the jails created by itself. It works only with official supported RELEASE(s). **It requires ZFS.**  
kjail does not spread files in the system. All stays in the same place (depending the pool you choose), except the kjail rc script which lands in /usr/local/etc/rc.d/.  
  
  
# 2) Installation  

Copy the files kinst and kjail.txz at the same place (this place has no importance). Then, chmod +x kinst and run it as root user.  
kinst will ask you on which pool you want to install kjail and then run the following commands (\*kpool\* = choosen pool):

`zfs create *kpool*/kjail`  
`zfs create *kpool*/kjail/base`  
`zfs create *kpool*/kjail/jails`  
`mkdir -p /*kpool*/kjail/settings/etc`  
`cp -p /etc/resolv.conf /*kpool*/kjail/settings/etc`  
`tar -xf kjail.txz -C /*kpool*/kjail`  
`sed -i "" -e "s#/\*\*\*/#/*kpool*/#" /*kpool*/kjail/kjail (adjustement of the location of kjail inside the rc script)`  
`cp /*kpool*/kjail/kjail /usr/local/etc/rc.d/`  



# 3) Deinstallation  

Run the following commands:

`# service kjail onestop`  
`# rm /usr/local/etc/rc.d/kjail`  
`# zfs destroy -rf *kpool*/kjail`  


# 4) Basic usage  

All commands need the root privilege to be run.
To execute a "command", you can run it with /\*kpool\*/kjail/"command" or service kjail one"command", if you didn't sysrc kjail_enable=YES. If you did, it's just service kjail "command".

To begin with, you need to load a RELEASE as base (see 5 principles for an explanation):  
`# /*kpool*/kjail/loadbase XX.x-RELEASE`  
Where XX.x is equal or inferior to the host RELEASE and is a supported RELEASE.  
Next, it's advised to update the base with the latest security patches:  
`# /*kpool*/kjail/update`  

After that, you can create any number of jail you want:  
`# /*kpool*/kjail/jcreate JailName`  
An ipv4 address and interface will be asked, but they are optional.  

You may want to edit the jail configuration file and/or the associated fstab:  
- /\*kpool\*/kjail/jails/JailName/JailName.conf
- /\*kpool\*/kjail/jails/JailName/JailName.fstab

You start the jail with:  
`# /*kpool*/kjail/start JailName`  

You get a command line into with:  
`/*kpool*/kjail/cons JailName`  

There, you can install the needed softwares, add users, modify configuration files and so on.  

If you want that this jail be launched automatically at boot, its name needs to be present in /*kpool*/kjail/kjail.conf, line jails_list="...". The question is asked at each jail creation. But, after creation, you just have to edit this file to add or remove jail name from the list. Note that jails will be run at boot only if there is kjail_enable=YES in /etc/rc.conf.  


# 5) Principles  

It's a nullfs based jail. There is one base for all jails. These jails use that base but in read-only mode. A nullfs mount is set between each jail (/\*kpool\*/kjail/jails/JailName/root/base) and /\*kpool\*/kjail/base. Soft links are set for numerous system directories (bin boot lib libexec rescue sbin usr/bin usr/include usr/lib usr/lib32 usr/libdata usr/libexec usr/ports usr/sbin usr/src usr/share).

Jails have obviously their own settings in /etc/ and their own softwares/settings in /usr/local.

Basically, when you update the base, you update all the jails. It's more or less the same thing concerning a RELEASE upgrade, although it's more complex in details.

All the jails resides in /\*kpool\*/kjail/jails/. When you create a jail, a new dataset \*kpool\*/kjail/jails/jailName is created, and is automatically mounted in the dir /\*kpool\*/kjail/jails/jailName. Inside this directory, you have JailName.conf which is the configuration file for this jail. You have also JailName.fstab where you can mount what you need for the jail. Finally, the /\*kpool\*/kjail/jails/jailName/root is the "/" of that jail.

When a jail is created, the whole content of /\*kpool\*/kjail/settings is copied in the root dir of this jail. At installation time (kinst), the file /etc/resolv.conf of the host is copied there in order to have a functional domain name resolution in each jail. But there are many things to add here, for instance: /etc/localtime, /etc/crontab, /etc/periodic.conf, /etc/rc.conf...

The change of RELEASE (upgrade) is a real problem for nullfs jails because this requires the update of the config files in each jail. To solve this, kjail uses etcupdate after the base upgrade has been completed. etcupdate needs the sources of the base system, and that's why loadbase fetches them.


# 6) kjail man  

- **bsnap**  
Manages snapshots concerning the base. Note that a snapshot named "ok" is automatically created before any update or upgrade. And the previous "ok" snapshot, if any, is deleted.  
&emsp;`bsnap [create|destroy|rollback] SnapshotName`  
&emsp;`bsnap list`  
All commands can be abbreviated like c* = create, d* = destroy, r* = rollback and l* = list. It's just the first letter that matters.
You can rollback only the most recent snapshot. If you need to rollback before that, you have to destroy the most recent snapshot(s) before.  

- **cons JailName**  
Launches a command line into the specified jail. It must be running. 

- **jcreate JailName**  
Creates a new jail. It's mainly interactive.

- **jdestroy JailName**  
Destroys the jail. This removes all its data.

- **jlist**  
Lists all the jails belonging to the kjail software. Those that are running and those that aren't.

- **jmend JailName**  
Tries to repair a jail after a RELEASE upgrade. Typically, that jail wasn't running during the upgrade process and needs at least an `etcupdate`.

- **jrename OldName NewName**  
Renames a jail.

- **jsnap**  
Manages snapshots concerning the jails. Note that a snapshot named "ok" is automatically created before any "etcupdate" (after the upgrade of the base). And the previous "ok" snapshot, if any, is deleted.  
&emsp;`jsnap [create|destroy|rollback] JailName SnapshotName`  
&emsp;`jsnap list JailName`  
All commands can be abbreviated like described for bsnap. Ditto concerning the rollback feature.

- **jup [JailName]**  
Performs a pkg upgrade on the jail JailName or on all jails that are running if no JailName is provided. A snapshot named "ok" is automatically created for the jail before pkg upgrade, and the previous "ok" snapshot, if any, is deleted.

- **loadbase XX.x-RELEASE**  
Downloads a RELEASE and its sources into /*kpool*/kjail/base/.

- **restart JailName**  
Runs a stop and a start of JailName.

- **start JailName**  
If this command is launched from the kjail service without argument, all jails belonging to the kjail starting list are started.

- **stop JailName**  
If this command is launched from the kjail service without argument, all jails belonging to the kjail system and currently running are stopped.

- **update**  
Performs an update (security patches) of the base system. You have to update the host system before. Once the base system is updated, this restarts all the kjail jails that were running.

- **upgrade XX.x-RELEASE**  
Performs an upgrade - minor or major - of the base system. The host system has to be at least equal to XX.x-RELEASE. Each running jail will execute `pkg-static upgrade -f` if this is a major RELEASE upgrade. In any case, all running jails will run "etcupdate" at the end of the process.

You can have some requests from `etcupdate resolve`. In general, you have to select (e) to edit the file, correct it with vi commands, and then (r) to indicate that the conflict has been resolved.
