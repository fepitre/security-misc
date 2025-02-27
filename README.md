# enhances misc security settings #

kernel hardening:

* deactivates Netfilter's connection tracking helper
Netfilter's connection tracking helper module increases kernel attack
surface by enabling superfluous functionality such as IRC parsing in
the kernel. (!) Hence, this package disables this feature by shipping the
/etc/modprobe.d/30_nf_conntrack_helper_disable.conf configuration file.

* Kernel symbols in /proc/kallsyms are hidden to prevent malware from
reading them and using them to learn more about what to attack on your system.

* Kexec is disabled as it can be used for live patching of the running
kernel.

* ASLR effectiveness for mmap is increased.

* The TCP/IP stack is hardened.

* his package makes some data spoofing attacks harder.

* SACK is disabled as it is commonly exploited and is rarely used.

* This package disables the merging of slabs of similar sizes to prevent an
attacker from exploiting them.

* Sanity checks, redzoning, and memory poisoning are enabled.

* The kernel now panics on uncorrectable errors in ECC memory which could
be exploited.

* Kernel Page Table Isolation is enabled to mitigate Meltdown and increase
KASLR effectiveness.

* SMT is disabled as it can be used to exploit the MDS vulnerability.

* All mitigations for the MDS vulnerability are enabled.

* The SysRq key is restricted to only allow shutdowns/reboots.
A systemd service clears System.map on boot as these contain kernel symbols
that could be useful to an attacker.

* Coredumps are disabled as they may contain important information such as
encryption keys or passwords.

* The thunderbolt and firewire modules are blacklisted as they can be used
for DMA (Direct Memory Access) attacks.

* IOMMU is enabled with a boot parameter to prevent DMA attacks.

* The kernel now panics on oopses to prevent it from continuing running a
flawed process.

Uncommon network protocols are blacklisted:
These are rarely used and may have unknown vulnerabilities.
/etc/modprobe.d/uncommon-network-protocols.conf
The network protocols that are blacklisted are:

* DCCP - Datagram Congestion Control Protocol
* SCTP - Stream Control Transmission Protocol
* RDS - Reliable Datagram Sockets
* TIPC - Transparent Inter-process Communication
* HDLC - High-Level Data Link Control
* AX25 - Amateur X.25
* NetRom
* X25
* ROSE
* DECnet
* Econet
* af_802154 - IEEE 802.15.4
* IPX - Internetwork Packet Exchange
* AppleTalk
* PSNAP - Subnetwork Access Protocol
* p8023 - Novell raw IEEE 802.3
* LLC - IEEE 802.2
* p8022 - IEEE 802.2

user restrictions:

* A systemd service mounts /proc with hidepid=2 at boot to prevent users from
seeing each other's processes.

* The kernel logs are restricted to root only.

* The BPF JIT compiler is restricted to the root user and is hardened.

* The ptrace system call is restricted to the root user only.

restricts access to the root account:

* Su is restricted to only users within the sudo group which prevents users
from using su to gain root access or switch user accounts.
/usr/share/pam-configs/wheel
(Which results in a change in /etc/pam.d/common-auth.)

* Logging into the root account from a virtual, serial, whatnot console is
prevented by shipping an existing and empty /etc/securetty.
(Deletion of /etc/securetty has a different effect.)
/etc/securetty.security-misc

access rights restrictions:

* The default umask is changed to 006. This allows only the owner and group
to read and write to newly created files.
/etc/login.defs.security-misc

* Enables pam_umask.so usergroups so group permissions are same as user
permissions. Debian by default uses User Private Groups (UPG).
https://wiki.debian.org/UserPrivateGroups
/usr/share/pam-configs/usergroups

* Removes read, write and execute access for others for all users who have
home folders under folder /home by running for example
"chmod o-rwx /home/user"
during package installation or upgrade. This will be done only once per folder
in folder /home so users who wish to relax file permissions are free to do so.
This is to protect previously created files in user home folder which were
previously created with lax file permissions prior installation of this
package.

access rights relaxations:

This package does (not yet) automatically lock the root account password.
It is not clear that would be sane in such a package.
It is recommended to lock and expire the root account.
In new Whonix builds, root account will be locked by package
anon-base-files.
https://www.whonix.org/wiki/Root
https://www.whonix.org/wiki/Dev/Permissions
https://forums.whonix.org/t/restrict-root-access/7658
However, a locked root password will break rescue and emergency shell.
Therefore this package enables passwordless resuce and emergency shell.
This is the same solution that Debian will likely addapt for Debian
installer.
https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=802211
Adverse security effects can be prevented by setting up BIOS password
protection, grub password protection and/or full disk encryption.
/etc/systemd/system/emergency.service.d/override.conf
/etc/systemd/system/rescue.service.d/override.conf

Disables TCP Time Stamps:

TCP time stamps (RFC 1323) allow for tracking clock
information with millisecond resolution. This may or may not allow an
attacker to learn information about the system clock at such
a resolution, depending on various issues such as network lag.
This information is available to anyone who monitors the network
somewhere between the attacked system and the destination server.
It may allow an attacker to find out how long a given
system has been running, and to distinguish several
systems running behind NAT and using the same IP address. It might
also allow one to look for clocks that match an expected value to find the
public IP used by a user.

Hence, this package disables this feature by shipping the
/etc/sysctl.d/tcp_timestamps.conf configuration file.

Note that TCP time stamps normally have some usefulness. They are
needed for:

* the TCP protection against wrapped sequence numbers; however, to
trigger a wrap, one needs to send roughly 2^32 packets in one
minute:  as said in RFC 1700, "The current recommended default
time to live (TTL) for the Internet Protocol (IP) [45,105] is 64".
So, this probably won't be a practical problem in the context
of Anonymity Distributions.
* "Round-Trip Time Measurement", which is only useful when the user
manages to saturate their connection. When using Anonymity Distributions,
probably the limiting factor for transmission speed is rarely the capacity
of the user connection.

Application specific hardening:

* Enables APT seccomp-BPF sandboxing. /etc/apt/apt.conf.d/40sandbox
* Deactivates previews in Dolphin.
* Deactivates previews in Nautilus.
* Deactivates thumbnails in Thunar.
## How to install `security-misc` using apt-get ##

1\. Add [Whonix's Signing Key](https://www.whonix.org/wiki/Whonix_Signing_Key).

```
sudo apt-key --keyring /etc/apt/trusted.gpg.d/whonix.gpg adv --keyserver hkp://ipv4.pool.sks-keyservers.net:80 --recv-keys 916B8D99C38EAF5E8ADC7A2A8D66066A2EEACCDA
```

3\. Add Whonix's APT repository.

```
echo "deb http://deb.whonix.org buster main contrib non-free" | sudo tee /etc/apt/sources.list.d/whonix.list
```

4\. Update your package lists.

```
sudo apt-get update
```

5\. Install `security-misc`.

```
sudo apt-get install security-misc
```

## How to Build deb Package ##

Replace `apparmor-profile-torbrowser` with the actual name of this package with `security-misc` and see [instructions](https://www.whonix.org/wiki/Dev/Build_Documentation/apparmor-profile-torbrowser).

## Contact ##

* [Free Forum Support](https://forums.whonix.org)
* [Professional Support](https://www.whonix.org/wiki/Professional_Support)

## Donate ##

`security-misc` requires [donations](https://www.whonix.org/wiki/Donate) to stay alive!
