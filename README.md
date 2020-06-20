# Scope
Instructions for various Unifi Dream Machine (UDM) and Unifi Dream Machine Pro (UDM-P) devices.  

The Unifi products have many instructions for the much older UGS and Unifi Cloudkey, both of which are functionally built into the UDM and UDM-P along with other Unifi products, but the directions are almost all out of date.
A primary difference is that the UGS and Cloudkey run an OS distribution that can easily have packages ported to it.  UDM/UDM-P don't run the same type of OS so this isn't possible.

All the instructions here will require you to log into your router manually via SSH at some point.

No changes here will survive a firmware update of the UDM/UDM-P, and will need to be repeated after each update.

# SSH Problems
Due to incredibly poor implementation, the "SSH Settings" available from the GUI configuration has absolutely no effect on anything.  The SSH login is always `root` and the password is always the primary administrator's password to log into the GUI.  This is your Unifi account password if you have Remote Management enabled.
