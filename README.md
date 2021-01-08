# Scope
Instructions for various Unifi Dream Machine (UDM) and Unifi Dream Machine Pro (UDM-P) devices.  

The Unifi products have many instructions for the much older UGS and Unifi Cloudkey, both of which are functionally built into the UDM and UDM-P along with other Unifi products, but the directions are almost all out of date.
A primary difference is that the UGS and Cloudkey run an OS distribution that can easily have packages ported to it.  UDM/UDM-P don't run the same type of OS so this isn't possible.

All the instructions here will require you to log into your router manually via SSH at some point.

No changes here will survive a firmware update of the UDM/UDM-P, and will need to be repeated after each update.

# SSH Problems
Due to incredibly poor implementation, prior to the 1.8.0 UDM release the "SSH Settings" available from the GUI configuration has absolutely no effect on anything.  The SSH login is always `root` and the password is always the primary administrator's password to log into the GUI.  This is your Unifi account password if you have Remote Management enabled.
If you have UDM 1.8.0 or later however, the settings can be properly configured in the specific Device configuration (not Network Settings, but the device specific settings where you can power off and/or reboot the device remotely).

# Downgrading & Backups
Backups backup the settings in the Network app of the UDM/UDMP, but do not back up the user accounts or device name (among other things).  If you factory reset, you will have to manually configure the name of the device again, and manually add all users to the device again.

Backups can only be restored to equal or newer versions of the Network app.  So if you downgrade the version of software in your unit, you probably won't be able to restore your most recent backup and will need to reconstruct your configuration from scratch.  However backups are forward compatible.  You can restore a backup from an older version on a newer software version.
Automatic backups contain the version number of the Network app they were generated from in their file names.  This can be handy in determining what version of software you can apply them to, but does require resolving a UDM/UDMP version number to its associated `Unifi Network` version number (available in the release notes https://community.ui.com/releases).

Downgrading a UDM/UDMP can be necessary if the system fails to function as expected after an upgrade.  It's important that you make sure you have the settings backed up before the upgrade since **you won't be able to retain settings during a downgrade or use backups created after you upgraded.**  The steps are rather simple, with one caveat at the end:
1. SSH to the terminal
2. Use the `ubnt-upgrade` console command to perform an "upgrade" to the older software version (find the URL in the release notes of the older version)
3. After the unit has "upgraded" and rebooted successfully, make a copy of your backups onto another machine (i.e. `scp`) if you haven't already
4. Perform a factory reset by either holding the power button for 10+ seconds (the light on the UDM will go white, then go out while you continue holding it), or by using the Device settings (not Network settings) and using the Factory Reset button under Advanced Settings.
5. After the unit has reset and rebooted, go thru the normal first-time setup instructions for the unit
6. At the end of the first time setup after you've selected/entered network speed numbers, the unit will do some additional "work" and then dump you into the usual screen with a button to go to the Network settings in the middle.  It will pop up with an overlay saying "Device Updating" on top of this screen.  You MUST pull the power plug from the unit at the start of this process (within about 10 seconds).  If you wait too long, it could be in the process of flashing itself instead of downloading or verifying the download and you may brick the unit.  If you don't pull power at all, it will automatically upgrade to the latest released version of software, regardless of what you told it about updating during the first time setup, and reboot itself again.
7. If you don't pull the plug on it to halt the forced update, you'll have to start over again with step 2.
8. In the Network Settings, under the Backup section, use the Restore button to pick a file from your computer to restore.

# Overview
The UDM and UDM-Pro run a custom variant of a Linux operating system.  The majority of the functionality is provided by a single large podman container though.   As a result, a number of configuration files are not directly modifiable.  To find where the important directories are, you can SSH into the device, get the name of the single large container (probably `unifi-os`)

```
podman container list
```
then get the information about it to see what mount points the container has:
```
podman container info unifi-os
```
Most of the mount points are of type `bind` which means they come from an actual folder external to the container.  One of the mountpoints is type `overlay` and has multiple directories specified.  This allows modifications to an underlying readonly mounted file system, but should only ever be accessed from the `work` directory to avoid corruption.
