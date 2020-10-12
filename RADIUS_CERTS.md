# Overview
The RADIUS server that comes built in to the UDM and UDM-Pro is a FreeRADIUS server that has been preconfigured.  Because it runs in a podman container, and some of the configuration files are modified automatically based on the WebUI, care should be taken when modifying the FreeRADIUS configuration.

# Certificates and RADIUS Background
Unlike most TLS certificates, RADIUS certificates should NOT be signed by a public root CA.  Doing so allows anyone with any certificate signed by a public root CA to spoof your RADIUS server's identity.
Resultantly, RADIUS certificates always have uniquely generated root CAs, and usually don't have intermediate CAs unless a number of separate RADIUS servers are needed within the network.

This also means that WiFi users of the WPA2 Corporate networks that use EAP with TLS, and optionally with TTLS and/or PEAP, will always need to import a private root CA for authenticating before sending credentials.

# Applicability
The UDM and UDM-Pro include RADIUS certificates, but it's unclear whether they're automatically generated on first use of the radius server (default behavior for freeradius) or pre-generated and included in the update images.  Given that the config files used by the FreeRadius tools to generate the certificates are not found in the system, it seems likely that the certificates were pre-generated and are included directly in the Ubiquiti firmware.  This would mean they're identical for all UDM and UDM-Pro devices (possibly all UDM Controllers) and are therefore NOT private like they need to be.  Additional evidence of this pre-generation is the fact that the instructions provided by FreeRadius for replacing the certificates with a previously generated set appear to be what's provided, with only the certs and keys present and not the config files or generator tools that would normally be present in the same directory.

These instructions assume that the certificates used by RADIUS by default are NOT secure and need to be replaced.  

## FreeRADIUS certificate generator
A fully installed but unconfigured instance of the FreeRadius dpkg is installed in `/usr/share/freeradius/`.  This is useful because the package contains extensive generation tools and READMEs for configuring it, specifically in the `/usr/share/freeradius/raddb/certs` folder.

While it's probably easiest to use the package already provided on the UDM/UDM-P, it is also an option to use any FreeRADIUS certificate generator on any system instead.

Regardless of which generator is used, be sure to securely back up all of your configuration files used when generating the certificates and keys, as well as the certificates and keys themselves.

# Instructions
TODO: should be based on instructions from `/usr/share/freeradius/raddb/certs/README`
