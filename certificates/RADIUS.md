# TL;DR
* RADIUS doesn't use internet-type certificates, it uses uniquely generated certificate authorities (CAs) and keys.
* Network users will need to receive their username, password, and a CA certificate. Jump down to [Using The Generated RADIUS CA](#using-the-generated-radius-ca) to see how to get the CA certificate you should distribute.
* UDM/UDM-P generates unique keys and certificates on first enabling of the RADIUS.
* Built-in UniFi cloud backup of the Network App includes the RADIUS keys and certificates so cloud restore recovers them.

# Background
The RADIUS server that comes built in to the UDM and UDM-Pro is a FreeRADIUS server that has been preconfigured.  Because it runs in a podman container, and some of the configuration files are modified automatically based on the WebUI, care should be taken when modifying the FreeRADIUS configuration.

## Certificates and RADIUS
Unlike most TLS certificates, RADIUS certificates should NOT be signed by a public root CA.  These certificates are used to uniquely identify your specific RADIUS server based on the public key they contain, unlike website certificates that are uniquely identified based on the hostname matching the name in the certificate.  Using globally public (i.e. internet) public key infrastructure (internet PKI) would allow anyone able to get any certificate for any website to spoof your RADIUS server.

In corporate installations where many different RADIUS servers may need to be present for different physical sites, an independent (non-internet) PKI may exist.  In these cases each RADIUS server may have a unique certificate signed by a root CA that's internal to the corporation only.  The certificate trust chain length (how many intermediate signing servers were used) may vary for this case, and the signing CA key(s) likely are not present on the RADIUS server.
For small scale and at-home users though, no such external PKI will usually exist.  In this case a unique set of keys is usually generated and retained on the RADIUS server itself and the only difficulty is in getting a copy of the root CA public key so it can be handed out to users.

This also means that WiFi users of the WPA2 Enterprise networks that use EAP with TLS, and optionally with TTLS and/or PEAP, will always need to import a private root CA for authenticating before sending credentials.  Client devices are designed to expect this use case, though many also provide a "Do Not Validate CA Certificate" to allow bypassing of it.  Except with certain specific devices and use cases, this should never be used.  

### Never Use "Do Not Validate CA Certificate"

Most WiFi clients that support WPA2 Enterprise include an option: "Do Not Validate CA Certificate".  This will cause the client to skip the step of verifying the RADIUS authentication server, and allows you to skip adding a certificate on your client device for it.  This is **extremely** insecure, and you're usually better off switching to WPA2 Personal in this case. 
Without CA Certificate verification of the RADIUS server, any WiFi that claims to have the same SSID as your WPA2 Enterprise network will receive your plaintext username and password.  Capturing this is a simple matter of an attacker setting up a WPA2 Enterprise access point (AP) with it's own RADIUS server, and turning on debug logging on the RADIUS server.  Your client will connect to that WPA2 Enterprise AP, receive a certificate from the RADIUS server that it never verified, then send your credentials to the RADIUS server thru the normal means.  Even the secure means of transfering credentials only avoid a thrid party seeing your plaintext username and password while it's on the wire, the RADIUS server itself will always have access to the plaintext version.

#### Caveat (Up-to-date Android 11 and later)
One caveat to this threat is certain, mostly very up-to-date Android, devices.  Around December 2020 Android pushed a security patch that initially disabled the ability for a user to choose not to validate the CA Certificates on WPA2 Enterprise networks. Pretty quickly thereafter they realized this blocked a large percentage of users from using WPA2 Enterprise networks, so they implemented an alternative. When a client device connects to a WPA2 Enterprise network for the first time, and gets a certificate from it that it hasn't seen before, it will prompt the user to see if they expected this WPA2 Enterprise network to be in this location.  If the user says "yes", then the received certificate is added to the list of allowed certificates for that SSID.  This allows the certificates to be checked and manually approved as trusted the first time, without requireing pre-distribution of them.

Care should be taken even with this case though.  This manual approval based on SSID being expected to be present is based on the assumption someone spoofing the WPA2 Enterprise SSID at the location of the real one while the user is connecting to it for the first time is extremely unlikely.  If you have public users regularly visiting and joining your network, this is no longer the case and the window of attack is present most of the time.  An attacker would have a tough time targeting a single specific user, but would have an easy time if they just wanted to target any possible user.

## UDM/UDM-P Applicability
The UDM and UDM-Pro generate a new CA key-pair and server key-pair the first time the FreeRADIUS server is started.  FreeRADIUS ships with a bootstrap.sh file that assists with doing exactly this, generating new unique-per-device key-pairs and automatically signing the generated server certificate with the generated CA key, though it's unclear whether the UDMs directly use this script or not.  

Because the UDM is automatically generating this without input from the GUI, they have some default configured.  The generated server certificates unconditionally contain the following metadata fields and values:
```
subject=/C=US/ST=New York/O=Ubiquiti Inc./CN=UbiOS RADIUS Server Certificate
issuer=/C=US/ST=New York/L=New York/O=Ubiquiti Inc./CN=UbiOS RADIUS Certificate Authority
```
There's nothing wrong with these values, it's perfectly fine for them to be present without it compromising any security.

### Where
Because UDM/UDM-P use podman containers to run everything, "standard" locations aren't always used.  FreeRADIUS expects the certificates in a specific location, but that location is within the container.  When viewed from outside the container this location may be entirely different.  

As is standard practice, the UDM/UDM-P locates the data for settings that need to be retained across reboots and updates in the /mnt/data/ partition.  The FreeRADIUS certificates are no exception.

After RADIUS has been enabled the first time from the WebUI, the new certificates and keys will be located in `/mnt/data/udapi-config/raddb/certs`.  Note that these will also appear to be in `/run/raddb/certs` and `/overlay/root_ro/usr/share/freeradius/raddb/certs`, but these are volatile locations related to the running of the container.

# Install/Use
After RADIUS has been enabled the first time from the WebUI, the new certificates and keys will be located in `/mnt/data/udapi-config/raddb/certs`.  Note that these will also appear to be in `/run/raddb/certs` and `/overlay/root_ro/usr/share/freeradius/raddb/certs`, but these are volatile locations related to the running of the container.

### Backup the Keys
It's unclear what causes the UDM/UDM-P to delete the old RADIUS certificates, but it's been seen after some software updates and Factory Reset.  

The built-in cloud backup functionality seems to preserve and restore these certificates, at least in 1.10.4 (good job on thinking of this Ubiquiti, the WiFi/network functionality wouldn't work as a seamless backup/restore otherwise).  Since the CA certificate is distributed to all network users though, it's very important these not be lost (or you'll have to re-distribute a new certificate to all your users).

Copy both the `*.key` and `*.pem` files to back them up.
```
mkdir my-radius-backup-dir
scp root@192.168.1.1:/mnt/data/udapi-config/raddb/certs/* ./my-radius-backup-dir/
```
This should be files:
```
ca.key
ca.pem
server-key.pem
server.pem
````
The files with word "key" in their name are private keys and need to be kept ultra-secret.  The other files are public key certificates and need no special protection against someone getting a copy of them.

### Restore Keys from Backup

As mentioned before, the built-on cloud backup functionality of the UDM/UDM-P includes (at least by 1.10.4) these keys and certificates.  Restoring from a UniFi cloud backup of the Network Application on the UDM/UDM-P will restore these as well.

If you want/need to restore the files themselves, the minimum that needs to be restored is the `server-key.pem` and `server.pem`.  Make sure these files were generated automatically by the UDM/UDM-P by enabling RADIUS in the WebUI first, then all the file and replace them with your backup copy.

*The names of the files used here are important and must be used on the UDM/UDM-P.*

1. Remove the existing ones:
```
ssh root@192.168.1.1
cd /mnt/data/udapi-config/raddb/certs/
rm ca.key ca.pem server-key.pem server.pem
```
2. Copy your backed up server key and certificate files
```
scp ./my-radius-backup-dir/server{-key,}.pem root@192.168.1.1:/mnt/data/udapi-config/raddb/certs/
```
3. (optional) Copy your backed up CA key and certificate files
```
scp ./my-radius-backup-dir/ca.{key,pem} root@192.168.1.1:/mnt/data/udapi-config/raddb/certs/
```
4. Reboot the UDM/UDM-P
5. Confirm the file on the UDM/UDM-P match what you restored

## Using The Generated RADIUS CA
*This applies to most users*

In this case the automatically generated keys and certificates will be used.  Since they are uniquely generated the first time FreeRADIUS is enabled, this is a perfectly reasoable option.  

The keys and certificates were already generated when the RADIUS server functionality was enabled the first time from the WebUI.  All that needs to be done is to get a copy of the CA certificate that was generated so it can be distributed to users.

Copy via SSH, the `ca.pem` file from the UDM/UDM-P to your machine so you can distribute it.
```
scp root@192.168.1.1:/mnt/data/udapi-config/raddb/certs/ca.pem ./my-radius-ca-cert.pem
````
You will need to enter your SSH password.  See the other READMEs in this repository for instructions on using SSH to access your UDM/UDM-P.

## Using a Newly Generated RADIUS CA and Key
*This is unnecessary, but included for completeness*

If for some reason you're unhappy with the perfectly secure automatically generated keys and certificates, you can follow the directions provided by FreeRADIUS to generate them on any machine, then copy them to the UDM/UDM-P.  

1. Generating these manually it outside the scope of this document, read about how to do it on the FreeRADIUS project
2. Follow the [Restore Keys from Backup](#restore-keys-from-backup) directions above

## Using an Enterprise Shared RADIUS CA
*This is a rather unique use-case and unlikely to apply to most users.*

If your use case requires you to use a RADIUS certificate signed by an externally shared CA, you will need to generate a server-specific key-pair, sign it with the appropriate CA, and replace the server key and server certificate in the UDM/UDM-P (see the [Restore Keys from Backup](#restore-keys-from-backup) directions above).

Instructions for generating a new key-pair and using an existing CA to sign it are left to the reader. Some details may be found in the [Advanced Details](#freeradius-certificate-generator).

The Shared RADIUS CA certificate is what is supplied to network users.

# Extra Advanced Details
Details here are for reference of advanced users and are not complete instructions.  

## FreeRADIUS certificate generator
A fully installed but unconfigured instance of the FreeRadius dpkg is installed in `/usr/share/freeradius/`.  This is useful because the package contains extensive generation tools and READMEs for configuring it, specifically in the `/usr/share/freeradius/raddb/certs` folder.

While it's probably easiest to use the package already provided on the UDM/UDM-P, it is also an option to use any FreeRADIUS certificate generator on any system instead.

Regardless of which generator is used, be sure to securely back up all of your configuration files used when generating the certificates and keys, as well as the certificates and keys themselves.
