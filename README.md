# xcp-ng
Helpful info for installing and managing XCP-ng, XO-Lite, and Xen Orchestra.

This document is a work in progress.  Just look at it! :persevere:
Use with caution!


[https://xen-orchestra.com/docs/]
[https://docs.xenserver.com/]
[https://docs.xcp-ng.org/]

[https://github.com/xcp-ng/]


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


### add certs to xcp-ng

1. add your root cert

Adding your root cert isn't required to connect to the XO Lite web interface but it will guarantee your server can talk to all your other servers without cert issues.

Certs and certificate authorities can be done in so many ways I can't possibly know how you will do it.  All I can say is you need the root cert, intermediate cert (if you use one), and the XCP-ng server cert and key.

# add your root cert to the trusted root certs
After the system reboots, SSH into the XCP-ng server as "root" and the password you provided during the install.

2. sudo vi /etc/pki/ca-trust/source/anchors/your-root-cert.crt
3. paste in your root certificate
4. update-ca-trust

Because update-ca-trust does not provide feedback, check the trust list to see if your root cert is in the list:
trust list | grep -i {a phrase from your cert}

For instance, to verify my development root cert was added: 
trust list | grep -i avegy

# 



# provision a cert for the XCP-ng and put it on the server
Per xx [https://docs.xcp-ng.org/guides/TLS-certificates-xcpng/] the full chain is optional but I recommend doing it anyway.  That means it should include the XCP-ng cert, the intermediate cert (if you have one), then the root cert.  Be sure each section is separated and they aren't running together.

1. CD the home folder or a safe location to store the cert files.  Your call.

SCP up or paste in the XCP-ng certifcate to cert.pem:
vi cert.pem

SCP up or paste in the XCP-ng private key to key.pem:
vi key.pem

SCP up or paste in the XCP-ng prcertificate chain to chain.pem:
vi chain.pem

Back up the existing SSL cert/chain file:
cp /etc/xensource/xapi-ssl.pem /etc/xensource/xapi-ssl.pem.bak

xe host-server-certificate-install certificate=<path to certificate> private-key=<path to key> certificate-chain=<path to chain>

xe host-server-certificate-install certificate=./cert.pem private-key=./key.pem certificate-chain=./chain.pem

Browse to the XCP-ng/XO-Lite URL and verify that the new certificate is being used. 
If you have any issues, try opening the URL in a private window.  My browsers sometimes take time to figure it out.

Test the certs:
https://{XCP-ng/XO-Lite URL}

If you have any issues, try opening the URL in a private window.  My browsers sometimes take time to figure it out.

# cert rollback
If the cert change just isn't working out, you can roll back to the XCP-ng/XO-Lite self-signed certs:
1. cp /etc/xensource/xapi-ssl.pem /etc/xensource/xapi-ssl.pem.broken
cp /etc/xensource/xapi-ssl.pem.bak /etc/xensource/xapi-ssl.pem

Reboot the server and test browsing to the site URL again.




### Update the XCP-ng server
<!-- After the system reboots, SSH into the XCP-ng server. -->
<!-- Log on as root and the password you provided during the installation. -->

According to cat /etc/os-release
ID_LIKE="centos rhel fedora"
so we need to use Yum to update the system:

yum update -y

Reboot the server when the update finishes. 

##############################
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

# Start the XOA VM:
xe vm-start vm="XOA"

# enable auto-start on the pool
1. list the pools
xe pool-list

2. Enaable auto-poweron on the pool
xe pool-param-set uuid=<UUID of the pool> other-config:auto_poweron=true


# enable autostart on the XOA server
1. Get the UUID of the XOA server:
xe vm-list

2. enable auto-start on the XOA server:
xe vm-param-set uuid=<VM_UUID> other-config:auto_poweron=true


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

#### Fix CA issues on XOA
Before logging into the web interface for the first time, you should add your root cert to the XOA server and provision a cert for the XOA server.

Adding your root cert isn't required to connect to the XOA web interface but it will guarantee your XOA server can talk to all your XCP-ng servers without cert issues.

Certs and certificate authorities can be done in so many ways I can't possibly know how you will do it.  All I can say is you need the root cert, intermediate cert (if you use one), and the XOA app cert and key.

## Add your own certificate authority and the XOA cer
# add your root cert to the trusted root certs
1. SSH into the XOA server as "xoa".
2. sudo vi /usr/local/share/ca-certificates/your-root-cert.crt
3. paste in your root certificate
4. sudo update-ca-certificates

Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.

# provision a cert for the XOA and put it on the server
Per xx [https://xen-orchestra.com/docs/configuration.html#https-and-certificates] the certificate.pem needs to be the full chain.  That means it should include the XOA cert, then intermediate cert (if you have one), then the root cert.  Be sure each section is separated and they aren't running together.

Note: If you want to look at the XOA configuration, it is located at:
/etc/xo-server/config.toml

1. sudo mv /etc/ssl/cert.pem /etc/ssl/cert.pem.bak
2. sudo mv /etc/ssl/key.pem /etc/ssl/key.pem.bak

Create a new cert.pem and paste in the full chain:
sudo vim /etc/ssl/cert.pem

Create a new key.pem and paste in the cert key:
sudo vim /etc/ssl/key.pem

Reload the daemon:
sudo systemctl daemon-reload


Restart the xo-server service:
sudo systemctl restart xo-server.service

Test the certs:
https://{XOA URL} or https://{XOA IP address}

If you have any issues, try opening the URL in a private window.  My browsers sometimes take time to figure it out.

# cert rollback
If the cert change just isn't working out, you can roll back to the XOA self-signed certs:
1. sudo mv /etc/ssl/cert.pem /etc/ssl/cert.pem.broken
2. sudo mv /etc/ssl/key.pem /etc/ssl/key.pem.broken
1. sudo mv /etc/ssl/cert.pem.bak /etc/ssl/cert.pem
2. sudo mv /etc/ssl/key.pem.bak /etc/ssl/key.pem
3. sudo systemctl daemon-reload
4. sudo systemctl restart xo-server.service

Test browsing to the site URL again.

#########################

# Log into the XOA dashboard
https://{XOA URL} or https://{XOA IP address}

If you have any cert issues you can always rename the new certs to pem.old, rename the old certs to .pem, and restart the 

Log on as the e-mail address and password you created earlier.

##
# register your XOA server
If you don't already have a Xen Orchestra account, you can create one here for free [https://xen-orchestra.com/]

From the XOA dashboard, enter your Xen Orchestra e-mail address and password in the registration box and register your XOA server.

# update your XOA software
Even though we updated the OS earlier, the XOA software likely needs to be updated.

Click on "upgrade" and follow the messages.  Be patient.  If the dashboard disconnects, wait a few minutes and refresh the browser.
You may need to upgrade a few times to get the XOA software completely up to date.

## add your first XCP-ng server to the pool
Click on Home >  Add Server

Provide the name, IP, root, and root password.
If you installed certs from your own internal CA, be sure to enable the Unauthorized Certificates.





####### mount nfs
Even if you will only have one XCP-ng server, you still need to add the host before you can add storage and VM's.




######### 
# list running vm's
xe vm-list

# shut down a vm
xe vm-shutdown name-label="XOA"

# check again before shutting down/rebooting xcp-ng server
xe vm-list


################
power_state:running 




use this later:
https://github.com/xcp-ng/win-pv-drivers/releases

