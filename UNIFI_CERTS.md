# Create a Certificate Singing Request
Follow standard online directions for generating a private key and creating a Certificate Signing Request (CSR) for a domain you control/own.

You will have a domain.key and domain.csr after this step, where domain.key is the private key and domain.csr is your signing request.

# Get a Signed Certificate
Use letsencrypt or whatever service.  Follow the standard directions for completing this.  It will use your domain.csr from the prior step.

The result of the certificate signing from the external service will be a signed certificate for your domain, and a "ca-bundle" that is the full certificate chain to reach your certificate provider.

You will have a domain.ca-bundle and domain.crt after this step, where domain.ca-bundle is the certificate chain without your certificate, and domain.crt is just your certificate.

# Create a fullchain
Create a combined certificate chain file.
```
cat domain.crt domain.ca-bundle > domain.fullchain.crt
```

You willl have a domain.fullchain.crt after this step.

# Create a PKCS12 file
In order to import into a Java keystore, you will need to create the binary-format .p12 file (PKCS12 format).

```
openssl pkcs12 -export -in domain.crt -inkey domain.key -certfile domain.fullchain.crt -out domain.p12 -name unifi -password pass:<somepassword> 
```

You will have a domain.p12 file after this step that is password protected and contains your private key, your public certificate, and the chain of trust from a root CA.

# SSH into the UDM/UDM-P
SSH into your router.  This is usually 192.168.1.1 and the username is always `root` (the SSH settings in the GUI do nothing at all).  The password is your admin user's password, which is your global Unifi password is you have remote login enabled.

# Locate the files
On your router, you'll need to locate the certificates and keys you're going to replace.  The exact location varies a little bit between versions, so you'll need to locate exactly where they are.

## Find the "cloudkey" files
```
find / -name "cloudkey.crt"
find / -name "cloudkey.key"
```
These will be in the same directory, likely something like /mnt/data/system/ssl/private/

From here on, these instructions will assume the files are at:
```
/mnt/data/system/ssl/private/cloudkey.crt
/mnt/data/system/ssl/private/cloudkey.key
```

## Find the "unifi-core" files
```
find / -name "unifi-core.crt"
find / -name "unifi-core.key"
```
These will be in the same directory, likely something like /mnt/data/unifi-os/unifi-core/config/

From here on, these instructions will assume the files are at:
```
/mnt/data/unifi-os/unifi-core/config/unifi-core.crt
/mnt/data/unifi-os/unifi-core/config/unifi-core.key
```

## Find Java keytool and keystore
Java uses its own keystore that is accessed via a `keytool` tool.

```
find / -name "keystore"
```
This is likely something like `/mnt/data/unifi-os/unifi/data/keystore`, which is what's assumed for the rest of these instructions.

The keytool is part of the Java tools.  On most Unifi systems there are multiple pods/containers that have Java in them.  They're all the same version (e.g. java-8-openjdk-arm64), so pick any one of them
```
find / -name "keytool" -path "*/jvm/*"
```
Since there are a bunch of options, these instructions will use the placeholder `/XXX/usr/lib/jvm/java-8-openjdk-arm64/bin/keytool`

# Backup the Unifi certs & keys
On your router, using the file paths found in the prior step
```
cp -a /mnt/data/system/ssl/private/cloudkey.crt{,.old}
cp -a /mnt/data/system/ssl/private/cloudkey.key{,.old}
cp -a /mnt/data/unifi-os/unifi-core/config/unifi-core.crt{,.old}
cp -a /mnt/data/unifi-os/unifi-core/config/unifi-core.key{,.old}
```

# Replace the Unifi certs & keys
From your PC, you'll scp the files to your router.  You'll need to enter the password to sign into your router on every one of these steps.

Be sure to replace the IP address with your router's, and the paths with those you found in the prior steps.
```
scp domain.fullchain.crt root@192.168.1.1:/mnt/data/system/ssl/private/cloudkey.crt
scp domain.key root@192.168.1.1:/mnt/data/system/ssl/private/cloudkey.key
scp domain.fullchain.crt root@192.168.1.1:/mnt/data/unifi-os/unifi-core/config/unifi-core.crt
scp domain.key root@192.168.1.1:/mnt/data/unifi-os/unifi-core/config/unifi-core.key
```

# Update the Java keystore
From your PC, you'll scp the p12 file to your router, then log into the router and run the keytool to update your keystore.

```
scp domain.p12 root@192.168.1.1:~
```
Then SSH into your router and run the following on it.  Be sure to use the p12 password you set when creating it as the `srcstorepass` 
```
cd ~
/XXX/usr/lib/jvm/java-8-openjdk-arm64/bin/keytool -importkeystore -deststorepass aircontrolenterprise -destkeypass aircontrolenterprise -destkeystore /mnt/data/unifi-os/unifi/data/keystore -srckeystore /root/domain.p12 -srcstoretype PKCS12 -alias unifi -srcstorepass <yourarchivepassword>
```
Replace the `unifi` entry when prompted by typing `yes` at the prompt.

The hardcoded keystore password for the router is `aircontrolenterprise`.  The hardcoded name is also `unifi`, you need to replace it.
As the root user, `~` is `/root`, which is where you copied the p12 file when you scp'd it onto the router.  Be sure to use the absolute path to your p12 file in this command.

# Set the Unifi controller to re-import, and restart it
A database version file is used within the router as an indicator file.  Removing it will cause re-parsing at next start.

Find it first:
```
find / -name "version" -path "*/db/*"
```
This is probably something like `/mnt/data/unifi-os/unifi/data/db/version`.

Remove it:
```
rm /mnt/data/unifi-os/unifi/data/db/version
```

Either reboot the UDM/UDM-P, or restart the process without rebooting:
```
unifi-os restart
```
