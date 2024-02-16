# xcp-ng
Helpful info for installing and managing XCP-ng and Xen Orchestra.


### download xcp-ng
you can download the latest stable release here:
https://xcp-ng.org/#easy-to-install

You can also browse the folder tree here:
https://updates.xcp-ng.org/isos/

#### 
Note: I had hardware issues with the latest stable release but was able to install the latest beta release.
If you get an error like "An unrecoverable error has occurred.  Failed to run efibootmgr." during the install it might be worth trying out the latest beta until the next stable release.

### install xcp-ng
The installation is very straightforward.  Boot to the ISO and answer all the questions.

Be sure and document the root password because you're going to need it all the time.

Reboot the server when ready.

Note: the "EFI_MEMMAP is not enabled." warning is not a big deal.  The Vates developers have said as much.

### Update the XCP-ng server
After the system reboots, SSH into the XCP-ng server.
Log on as root and the password you provided during the installation.

According to cat /etc/os-release
ID_LIKE="centos rhel fedora"
so we need to use Yum to update the system:

yum update -y

Reboot the server when the update finishes. 

### install xen orchestra (XOA)
Vates provides several methods  [https://xen-orchestra.com/docs/installation.html] to install XOA.
https://xen-orchestra.com/docs/xoa.html#alternative-install

Method 1 - Deploy XOA using their web-based installer

This method requires you to have an internet connection to visit their website and launch the install.

Before launching their installer, you need to browse to the IP address of the XCP-ng server and accept the SSL certificate.  Otherwise the XOA installer will hit a snag because of the untrusted SSL certificate.  If you don't do this in advance the XOA installer will tell you to do it so why not get that out of the way?

After you browse to the XCP-ng server, visit the  [https://vates.tech/deploy/] page and start the installation.



Method 2 - Deploy XOA using an XVA import (Preferred)

This is my preferred method because you are never providing your credentials on the website.  I have zero reason not to trust Vates.  I don't trust anyone!  Plus if you want to install in an air-gapped environment this is going to be your best bet.

You can read about this method here  [https://xen-orchestra.com/docs/xoa.html#via-a-manual-xva-download]

Download the latest XVA from here [https://xen-orchestra.com/products/unified]

Note: I tried to wget the file right on the server but the real URL is hidden away behind a bunch of JS.

SCP the file up to the XCP-ng server as root.

SSH into the XCP-ng server again as root.

Import the XVA:
xe vm-import filename=xoa_unified.xva

After the import completes, list the available VM's:
xe vm-list

Start the XOA VM:
xe vm-start vm="XOA"

Get the UUID of the XOA server:
xe vm-list

# Set the "xoa" user SSH password:
xe vm-param-set uuid=<UUID> xenstore-data:vm-data/system-account-xoa-password=<password>

For example: 
xe vm-param-set uuid=aabb1122-112a-3096-004f-830bbb038fc4 xenstore-data:vm-data/system-account-xoa-password='MyPassW0rd!'

# Reboot the XOA server:
xe vm-reboot vm="XOA"

# Get the DHCP assigned IP of the XOA server:
xe vm-list params=networks

# SSH into the XOA as "xoa" and the password you set a few steps ago.

# Set a static IP (optional):
xoa network static

As soon as the new IP applies you will need to SSH into the new IP as "xoa" again.

# Create an account for the web interface
sudo xo-server-recover-account youremail@here.com

# update the XOA server
According to cat /etc/os-release
ID=debian
so we need to use apt to update the system:

sudo apt-get update 
sudo apt-get dist-upgrade (sudo apt-get upgrade if you prefer)

Reboot the server when the update finishes. 

# Log into the XOA web interface
https://{XOA IP address}

Log on as the e-mail address and password you created earlier.


################

