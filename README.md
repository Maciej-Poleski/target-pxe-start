# Target PXE start

A tool combining WoL, proxyDHCP, TFTP server and PXE enabling remote start of selected target machine using chosen boot image.

## Critical user journey

* You have a NAS (Network Attached Storage) in your network (or you can use VPN).
* It's encrypted - both data as well as it's own software is encrypted at rest.
* Due to the above, encryption is protecting your data only when NAS is powered off.
* You also want to save on electricity and not run the NAS when nobody uses it, for example during nights.
* Encryption key is stored separately - bundled with Operating System kernel (for example in initramfs image) and stored on client machines (machines which are supposed to have access to data stored in NAS).
  * Client machines vary from PC through laptops to smartphones.
  * Image can be stored as a plain file on encrypted partition (PC) or encrypted file using hardware supported key chain for key retrievel (Smartphone).
* Maybe you also have a separate machine - backup. Organized in a similar way - fully encrypted.
* A client can request NAS to be:
  1. Remotely powered on (using Wake on Lan)
  2. Pointed at client machine to fetch boot image (using proxyDHCP) .
  3. Start local TFTP server serving selected boot image to (only) selected target machine.
  4. Shut down TFTP and proxyDHCP as soon as the target machine has been booted.

The procedure should require minimal configuration and permissions. Preferably you only need to specify boot image and target MAC address and can run the tool using unprivileged local accout.

## Architecture

The tool should eventually support three platfroms: Linux, Windows and Android. Support for Linux has priority. This repository contains only proof of concept and prototype. The desired solution consists of a few interchangeable modules enabling support and optimal code reuse across different platforms.

* *UI/command line* gathers required configuration from the user and provides information about progress of remote boot process.
* *Wake on Lan* issues [WoL](https://en.wikipedia.org/wiki/Wake-on-LAN) magic packet to selected target machine.
* *[proxyDHCP](https://en.wikipedia.org/wiki/Preboot_Execution_Environment#Integration)* issues `DHCPOFFER` packet with details about PXE environment such as address of local TFTP server serving selected boot image.
* *[TFTP](https://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol) Server* sends the boot image (only) to the target machine.
* *Image provider* provides boot image for the TFTP server to be sent to the target machine.

While the general architecture and business logic remains the same across supported platforms, some modules might require dedicated implementation. It is especially foreseen that Android will require dedicated UI and most likely [KeyChain](https://developer.android.com/reference/android/security/KeyChain) based image provider (Smartphone is exceptionally vulnerable to unauthorized access attempts while the NAS encryption key has to be kept safe all the time). Hopefully network libraries can be implemented in a cross-platform way.
