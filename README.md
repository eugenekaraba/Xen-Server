# Xen-Server
Instructions On Xen Server Installation
Initial Xen server installation: https://wiki.xenproject.org/wiki/Xen_Project_Beginners_Guide
youtube video on the topic: https://www.youtube.com/watch?v=n3POmlMNzvw

0. Install and configure xen-tools
apt-get install xen-tools

To give a different path where the domU images being saved and enable the superuser password in the initial build, we will edit the /etc/xen-tools/xen-tools.conf file and uncomment this lines:
dir = /home/xen/
passwd = 1

1. Create PV guest:
xen-create-image --hostname=tutorial-pv-guest --memory=512mb --vcpus=2 --lvm=vg0 --dhcp --pygrub --dist=stretch

This command instructs xen-create-image (the primary binary of the xen-tools toolkit) to create:
a guest domain with 512MB of memory, 
2 vcpus, 
using storage from the vg0 volume group we created, 
use DHCP for networking, 
pygrub to extract the kernel from the image when booted and 
lastly we specify that we want to deploy a Debian stretch operating system.

2. Starting a console guest

xl create -c /etc/xen/tutorial-pv-guest.cfg

which is translated into those 2 commands because of the -c (Attach console to the domain as soon as it has started.  This is useful for determining issues with crashing domains and just as a general convenience since you often want to watch the domain boot.):

xl create /etc/xen/tutorial-pv-guest.cfg && xl console tutorial-pv-guest

if you wanna go back to dom0 guest once you are logged in a domU guest press ctrl+]

3. Shutdown a guest

xl shutdown tutorial-pv-guest

PV Guests:
tutorial-pv-guest:
username: root, password: evgenios
username: evgenios (sudoer), password: evgenios

tutorial-pv-guest2:
username: root, password: evgenios2
username: evgenios2 (sudoer), password: evgenios2

tutorial-pv-guest3:
username: root, password: evgenios3
username: evgenios3 (sudoer), password: evgenios3

tutorial-pv-guest4:
username: root, password: evgenios4
username: evgenios4 (sudoer), password: evgenios4

tutorial-manual-pv-guest:
username: root, password: evgeniosmanpv
username: evgeniosmanpv (sudoer), password: evgeniosmanpv

4. Manually Create PV guest (Debian from netboot installation)

Download from the Dutch mirror: http://ftp.nl.debian.org/debian/ (hehe, Go Twente!!! let's see TU/e doing hosting a mirror)

Download vmlinuz and initrd.im (and debian.cfg) from here: http://ftp.nl.debian.org/debian/dists/Debian9.6/main/installer-amd64/current/images/ choose cdrom if you wanna install from the iso or netboot if you wanna install from the net installer

important link: http://ftp.debian.org/debian/dists/stretch/main/installer-i386/current/images/cdrom/xen/debian.cfg
important link (Xen man pages): https://wiki.xenproject.org/wiki/Xen_Man_Pages

step 1: mkdir -p /scratch/debian/stretch/netboot/amd64
step 2: cd /scratch/debian/squeeze/amd64
step 3: wget http://ftp.nl.debian.org/debian/dists/Debian9.6/main/installer-amd64/current/images/netboot/xen/debian.cfg
step 4: wget http://ftp.nl.debian.org/debian/dists/Debian9.6/main/installer-amd64/current/images/netboot/xen/initrd.gz
step 5: wget http://ftp.nl.debian.org/debian/dists/Debian9.6/main/installer-amd64/current/images/netboot/xen/vmlinuz
step 6: modify the debian.cfg file
	kernel = "/scratch/debian/stretch/netboot/amd64/vmlinuz"
	ramdisk = "/scratch/debian/stretch/netboot/amd64/initrd.gz"
	extra = "debian-installer/exit/always_halt=true -- console=hvc0" --> what does this thing do?????
	name = "tutorial-manual-pv-guest"

	memory = 512

	disk = ['phy:/dev/vg0/tutorial-manual-pv-guest,xvda,w']
	# vif = ['mac=00:16:3e:00:00:11, bridge=xenbr0']
	vif = [' ']

	CHECK THE STANZAS IN THE debian.cfg to see what are the configurations meanings (check http://xenbits.xen.org/docs/unstable/man/xl.cfg.5.html#HVM-guest-options for more information)

step 7: lvcreate -L 4G -n tutorial-manual-pv-guest /dev/vg0
step 8: mv debian.cfg tutorial-manual-pv-guest.cfg
step 9: ln -s /scratch/debian/stretch/netboot/amd64/tutorial-manual-pv-guest.cfg /etc/xen/tutorial-manual-pv-guest.cfg
step 10: xl create -c /etc/xen/tutorial-manual-pv-guest.cfg
step 11: xl destroy tutorial-manual-pv-guest (???? --> maybe the fact that I fixed the debian-installer/exit/always_halt=true -- console=hvc0 command does not require to destroy the guest TBD)
step 12: Once installation has completed the guest will shutdown and exit. At this point you should remove the kernel, initrd and extra lines from the guest configuration and replace them with simply:

	bootloader = "pygrub"

step 13: xl create -c /etc/xen/tutorial-manual-pv-guest.cfg

5. Create HVM guest (Ubuntu from cdrom installation) --> MUCH HARDER THAN I THOUGHT. THERE IS NO ONE RECIPE FITS ALL. EACH OS REQUIRES DIFFERENT PARAMETERS

Finally something that makes sense: https://help.ubuntu.com/community/XenProposed

step 1: mkdir -p /scratch/ubuntu/18-04/cdrom/amd64
step 2: cd /scratch/debian/stretch/cdrom/amd64
step 3: wget http://releases.ubuntu.com/18.04.1/ubuntu-18.04.1-desktop-amd64.iso

lvcreate -L 4G -n ubuntu-hvm /dev/vg0
touch ubuntu-hvm.cfg
	builder = "hvm"
	name = "ubuntu-hvm"
	memory = "512"
	vcpus = 1
	vif = ['']
	disk = ['phy:/dev/vg0/ubuntu-hvm,xvda,w','file:/scratch/debian/stretch/cdrom/amd64/ubuntu-18.04.1-desktop-amd64.iso,xvdc:cdrom,r']
	vnc = 1
	boot="dc"

xl create /etc/xen/ubuntu-hvm.cfg

nice explanation of the parameters: https://www.virtuatopia.com/index.php/Configuring_and_Installing_a_Xen_Hardware_Virtual_Machine_(HVM)_domainU_Guest


IMPORTANT COMMANDS

list all images: xen-list-images
delete an image: xen-delete-image <image_name>
remove logical volume: lvremove <path of the logical_volume> (more on debian lvm here: https://wiki.debian.org/LVM#List_of_VG_commands)
lists all the logical volumes: lvs
shutdown a guest: xl shutdown <guest_name>
report information about physical (and logical) devices: pvs -a
start a console to a guest: xl console <name_of_the_guest>, enter
xl list
brctl show




INTERESTING LINKS (had to close them down because of connectivity issues):
https://www.xenproject.org/users/getting-started.html (Getting Started)
https://wiki.xenproject.org/wiki/Xen_Project_Beginners_Guide (Xen Project Beginners Guide)
https://wiki.xenproject.org/wiki/Debian_Guest_Installation_Using_Debian_Installer (Debian Guest Installation Using Debian Installer)
https://wiki.debian.org/BridgeNetworkConnections (BridgeNetworkConnections)
https://wiki.xen.org/wiki/Network_Configuration_Examples_(Xen_4.1%2B) (Network Configuration Examples (Xen 4.1+))
https://wiki.debian.org/NetworkConfiguration (NetworkConfiguration)
https://www.howtoforge.com/how-to-run-fully-virtualized-guests-hvm-with-xen-3.2-on-debian-lenny-x86_64 (How To Run Fully-Virtualized Guests (HVM) With Xen 3.2 On Debian Lenny (x86_64))
https://www.howtoforge.com/virtualization-with-xen-on-debian-lenny-amd64 (Virtualization With Xen On Debian Lenny (AMD64))
https://wiki.xen.org/wiki/Debian_Guest_Installation_Using_Debian_Installer (Debian Guest Installation Using Debian Installer)
https://help.ubuntu.com/community/XenProposed (XenProposed)

https://www.virtuatopia.com/index.php/Configuring_and_Installing_a_Xen_Hardware_Virtual_Machine_(HVM)_domainU_Guest (Configuring and Installing a Xen Hardware Virtual Machine (HVM) domainU Guest)
https://lzone.de/cheat-sheet/Xen (Xen cheat sheet)


https://www.systutorials.com/241161/xen-hvm-domu-configuration-file/

Tmux Commands
create a new session: tmux new -s <name_of_the_session>
attach to a session: tmux attach -t <name_of_the_session>
list all sessions: tmux ls
dettach from a session: ctrl+b, d

Ipfire on HVM
https://wiki.ipfire.org/virtualization/xen/hvm-on-debian
https://docs.gns3.com/appliances/ipfire.html (scon ipfire images can be downloaded from here)


XEN NETWORKING (BRIDGING etc.)
xen networking examples: https://wiki.xenproject.org/wiki/Network_Configuration_Examples_(Xen_4.1%2B)
xen networking: https://wiki.xenproject.org/wiki/Xen_Networking
brctl command for bridging manipulation: https://www.tldp.org/HOWTO/BRIDGE-STP-HOWTO/set-up-the-bridge.html
debian bridging network connections: https://wiki.debian.org/BridgeNetworkConnections

When I do: brctl addif xenbr2 wlp8s0 I get: can't add wlp8s0 to bridge xenbr2: Operation not supported
Possible solution: iw dev wlp8s0 set 4addr on ----> YEAH IT WORKED!!!!! (https://wireless.wiki.kernel.org/en/users/documentation/iw) but in case there will be problems in the future check this: https://askubuntu.com/questions/155041/bridging-loosing-wlan-network-connection-with-4addr-on-option-why
I also need to assign the IP's to the xenbr1 (for the green LAN) and to the xenbr2(for the blue LAN) manually at the /etc/network/interfaces file


Swap Memory
https://www.digitalocean.com/community/questions/how-to-change-swap-size-on-ubuntu-14-04

Help me make sense of Citrix XenServer vs xenserver.org

-Xen: the basic hypervisor technology

-XenSource: the original company which was later acquired by Citrix

-The Xen Project: what Xen became after the project itself was moved control of The Linux Foundation

-XenServer: Citrix's own implementation of the Xen hypervisor in to a standalone platform, which is still a FOSS project but also has an enterprise supported version, which is what Citrix sells
