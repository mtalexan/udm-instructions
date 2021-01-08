# How
Backups are primarily accomplished thru the built-in tools in the Network Settings.  A Backup section of the Settings contains options for 
* Creating automatic backups on a schedule
* Restoring backups via upload
* Modifying the contents of backups
* Downloading individual backups
* Creating backups on demand
Backups are created in the local storage of the UDM/UDMP and must be manually moved/downloaded to alternative storage.

An alternative is the [BackiFi](https://backifi.com) free service.  This is a third-party tool that allows you to automate logins to your UDM/UDMP from their server that generate and download backups using the "manual" option, and then upload those files to a cloud storage provider you specify/own.

# Console
Backups are related to the Network app, so they are under the data folder associated with it.  When SSH-ed into the device, all automatically generated backups are found at `/mnt/data/unifi-os/unifi/data/backups/autobackup/`.
Replacing or modifying a set of automatic backups to this directory so that they re-appear in the GUI doesn't seem to be possible at this time.
The files created by scheduled automatic backups all contain the version of the Network app they were generated from, and the UTC+0 `YYYYMMDD_HHMM` timestamp (it doesn't matter what your device's timezone is set to). 

Creating backups from the command-line: *TBD*

# Limits
Backups contain the settings for the Network application, which is the majority of what you can modify on a system.  Notably it does not contain:
* The name of the UDM/UDMP device
* The users associated with the device, either a Unifi account or users with local credentials
* Device remote access settings
* Device SSH password
* Certificates added/modifed in the system
* *TBD*

Backups can only be restored on equal or newer versions of the `Unifi Network` component (the Network app).  Attempting to restore a backup generated on a newer version of the Network app will reject the backup.  

# Restoring
Restoring from a backup is accomplished from the Network Settings menu under the Backup section.  Clicking the "Restore from backup" button will prompt you to select a backup file from your local computer to upload, and will then begin restoring from it.  Restoring overwrites all current Network settings with the contents of the backup file.  

Restoration occurs atomically, so a failed restoration will not leave your system in a partially restored state.

After a Factory Reset, you must complete the First Time Setup wizard before you are able to access the Restore settings.  Be aware of the [limitations](Limits) of the backups, there are some parts of the First Time Setup that are not overwritten by a restore.
