# Upgrades 
Upgrades can be configured to occur automatically, triggered from the Device Settings (not Network Settings) manually, or performed manually from the SSH console with an HTTP or HTTPS firmware link.

## Limitations
Be aware of the upgrade limitations posted in the forums with each release.  Official candidates only infrequently have restrictions on what version you can apply them to, but Beta and Release Candidates (RCs) often have more.  

***It's not clear whether the automatic updates or manually triggered upgrades are aware of the upgrade restrictions, so be careful with setting the Beta or Release Candidate channels*** 

## Beta vs Release Candidate vs Official Channels
Joining the beta channel is quick, easy, and free.  It comes with access to additional community forums, and access to an update channel selector dropdown in the Device Settings.  However it also requires you to agree to extensive Terms of Service that include not talking about the contents of the Beta/RC releases.  These TOS are actually enforced, and have resulted in some Youtubers having action taken against them when they weren't very careful about what they said (supposedly).
Sign up is available [here](http://www.ubnt.com/beta) and grants immediately access to the beta community forums.

Once you are part of the Beta community, an additional drop-down selector will become available in your UDM/UDMP Device Settings (not Network Settings), though it remains set to the Official channel by default.  Modifying this value controls what software versions are considered to be available for upgrade, affecting both automatic updates and whether/when the manual update link in the Device Settings is displayed.

Unless you have joined the Beta community, the dropdown channel selector is not visible and is permanently set to "Official".

## Checking for new versions
Check for available updates occurs based on a configured check frequency in the Device Settings.  If a newer version than what is currently running is found for the selected [update Channel](Beta vs Release Candidate vs Official Channels), it is identfied as being available and can be applied via [automatic update](Automatic Updates) or [manually triggered upgrade](Manually Triggered Upgrades).

## Automatic Updates
Automatic updates can be enabled or disabled, and a frequency set for how often new updates are checked/applied.  During First Time Setup, this is a single drop down that lets you pick to disable or a check frequency, but afterward you're able to separately modify the check frequency and whether the updates are applied automatically or not.

Enabling automatic updates will automatically apply the updates at *TBD* time in the local timezone after a new update is [discovered](Checking for new versions).  

## Manually Triggered Upgrades
Manually triggered upgrades are available when a [new version is found](Checking for new versions) and either [automatic updates](Automatic Updates) are disabled, or the time window during which they are applied has not yet been reached.
When an upgrade is available for manual triggering, a hyperlink is shown below the version number in the Device Settings informing you that an upgrade is available.  Clicking it will prompt you to begin applying it.

## Via SSH Console
When new firmware is released on the [forums](https://community.ui.com/releases), it includes the directions for applying it from the SSH console, and HTTPS hyperlinks the UDM and UDMP firmware can be found at.  Logging into the UDM/UDMP via SSH and running the commands as instructed will automatically download and apply the upgrade, including rebooting the device.

# Downgrading
Downgrading is strongly discouraged due to the requirements in performing it, the potential for bricking your device, and the high likelihood you will have to reconfigure your network from scratch.  However it's sometimes the only solution when you have applied an upgrade that doesn't work.

# Limitations
Downgrading is allowed, but just like 

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
