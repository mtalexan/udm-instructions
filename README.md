# Scope
Instructions for various Unifi Dream Machine (UDM) and Unifi Dream Machine Pro (UDM-P) devices.  

The Unifi products have many instructions for the much older UGS and Unifi Cloudkey, both of which are functionally built into the UDM and UDM-P along with other Unifi products, but the directions are almost all out of date.
A primary difference is that the UGS and Cloudkey run an OS distribution that can easily have packages ported to it.  UDM/UDM-P don't run the same type of OS so this isn't possible.

All the instructions here will require you to log into your router manually via SSH at some point.

No changes here will survive a firmware update of the UDM/UDM-P, and will need to be repeated after each update.

# SSH Problems
Due to incredibly poor implementation, prior to the 1.8.0 UDM release the "SSH Settings" available from the GUI configuration has absolutely no effect on anything.  The SSH login is always `root` and the password is always the primary administrator's password to log into the GUI.  This is your Unifi account password if you have Remote Management enabled.
If you have UDM 1.8.0 or later however, the settings can be properly configured in the specific Device configuration (not Network Settings, but the device specific settings where you can power off and/or reboot the device remotely).

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
