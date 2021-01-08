# Upgrades 
Upgrades can be configured to occur automatically, triggered from the Device Settings (not Network Settings) manually, run manually with HTTP firmware links from the Network -> Devices page under the UDM/UDMP device's -> Settings -> Maintenance section, or performed manually from the SSH console with an HTTP or HTTPS firmware link.

# Beta vs Release Channels
Joining the beta channel is quick, easy, and free.  It comes with access to additional community forums, and access to a update channel selector dropdown in the Device Settings.  However it also requires you to agree to extensive Terms of Service that include not talking about the contents of the Beta releases.  These TOS are actually enforced, and have resulted in some Youtubers having action taken against them when they weren't very careful about what they said (supposedly).
Sign up is available [here](http://www.ubnt.com/beta) and grants immediately access to the beta community forums.

Once you are part of the Beta community, an additional drop-down selector will become available in your UDM/UDMP Device Settings (not Network Settings), though it remains set to the Release channel by default.  Modifying this value controls what software versions are considered to be available for upgrade, affecting both automatic updates and whether/when the manual update link in the Device Settings is displayed.

# Automatic Updates


Blah Blah TODO

When enabled, the automatic updates will always follow the Release channel unless you have signed up for the Beta community .  Only if you have Beta channel access with the account that is the owner of the UDM/UDMP will you have a drop down present in the Device Settings that allows you to configure which channel you want to use for automatic updates.
The Update Channel setting is used, if present, even when automatic updates are not enabled to pick the "newest" version of software available.  When there is a newer version of software than the one you're running available, a link below the current version number in the Device Settings will be present telling you so.  Clicking the link will automatically upgrade to the newest available version in your selected channel.




# Downgrading & Backups

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
8. On some/most versions, halting this automatic upgrade also leaves the Device SSH enabled with a default password (normally it gets disabled and cleared).  You must manually disable this or change the password from Device Settings (not Network Settings).
9. Change any of the other Device Settings.
8. In the Network Settings, under the Backup section, use the Restore button to pick a file from your computer to restore.
