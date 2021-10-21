# eff_you_os
How to abuse auto-boot and the M1 FUOS to create a bootkit

# Apriori

* https://github.com/AsahiLinux/m1n1
* https://github.com/AsahiLinux/docs
* https://github.com/mwpcheung/AppleSEPFirmware

## iBoot on the Mac

When Apple brought macOS to the M1 the community was immediatly concerned that the mac would suffer lock in to the operating system like the iDevice ecosystem.  Apple solved this problem with a clever signing system that allows an authorized adiministrator to modify the database to run a "fully unsigned OS" (backrynim by the community).  This allowed a hash of a next stage to be selected as the bootloader.

## Reviving the Dark Wake attacks

Being able to run any payload, combined with `auto-boot=true` allows for a malicious payload to be executed before the macOS loader.  Because there is no visual indication that the system is booting a FUOS target, a victim may be completely unaware that the copy of macOS being booted may have been tampered with by earlier boot stages.  Combine this with a malware kit that immedatly goes to a hibernate state, having hooked the system then is in control of the boot when the power button is pressed.

## Terminate and Stay Resident

By hooking the boot loader, and by using the WDT / Alarm system to occasioanlly wake and respond, the bootkit has control of the early stages of boot.  Combine this with the GPIO control of the LED and it's color (made obvious by the fact that recovery can in fact cause a S-O-S blink pattern showing it is under operating system control) as well as responding to a long press before the system does, thereby socially engineering the user to remove the press before the system fully recognises a gesture, persistence may be maintained.

## Chainloading iBoot and macOS / XNU

By making the bootkit modify the security properties of the operating system, and then booting XNU the bootkit breaks the secure boot process.

## Avoiding 1TR

* First break 1TR
* Repond to a long press before the WDT

## Faking the SEP for macOS

Emulating a SEP by changing the DeviceTree addresses or using a EL2 mapping scheme allows for the bootkit to intercept all commands destined for the SEP.  Looking for the existing implementation of a virtualized SEP implmenting the same mailbox and commands, which I had previously seen open source (please PR if you find it!)

# But Wait, Local Access, Admin Password?

* Local events occur and if a valuable target happen often
* One bad sholder surf, poorly placed security camera, or fake `sudo`

## Countermeasures

### Detection (WIP, dependent on LIMD changes)

Booting to DFU then Recovery and using `getenv` to query the machine state

### Removal

Booting to DFU via `macvdmtool` is a high integirty way to enter DFU mode, a clean technician workstation is then able to reinstall UniversalMac.ipsw.  This is unfortantly a destructive process for the data partition.

### Apple Design Change

#### Insecure Boot Indicator

By having a mac configured to boot FUOS first bring up the graphics, show a visual indicator (such as a unlocked padlock with a support link) and pausing a number of seconds (similar to how Chrome OS devices indicate a non-secure boot path) the device owner would be aware of such a modification.  To be fair this is the same issue that has existed with the T2 for it's history.  A lack of indicator when a device is being booted with less then full security or to an update / recovery (and therefore SIP is not protecting the system)

#### Seperate Credential

This is analagous to how Apple handled firmware security in the past.  While the user day-to-day could use a local admin password, a seperate firmware password prevented that from escilating to a comprimise of the plaform security state.  With the M1 the admin password was merged with the platform security password.  These could again be seperated for higher security scenerios.
