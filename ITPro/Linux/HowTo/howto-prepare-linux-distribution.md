<properties linkid="manage-linux-howto-linux-agent" urlDisplayName="Prepare a distribution" pageTitle="Prepare a distribution of Linux in Windows Azure" metaKeywords="Azure Git CodePlex, Azure website CodePlex, Azure website Git" metaDescription="Learn how to use Git to publish to a Windows Azure web site, as well as enable continuous deployment from GitHub and CodePlex." metaCanonical="" disqusComments="1" umbracoNaviHide="1"  writer="kathydav" editor="tysonn" manager="jeffreyg"/>

<div chunk="../chunks/linux-left-nav.md" />

# Prepare a Linux VM for Azure 

A virtual machine that you create in Windows Azure runs the operating system that you choose from the supported operating system versions. You can customize the operating system settings of the virtual machine to facilitate running your application. The configuration that you set is stored on disk. You create a virtual machine in Windows Azure by using a virtual hard disk (VHD) file. You can choose to create a virtual machine by using a VHD file that is supplied for you in the Image Gallery, or you can choose to create your own image and upload it to Windows Azure in a VHD file.

The following resources must be available to complete this task:

- **Server running the Windows Server operating system.** This task depends on using the Hyper-V Manager that is a part of the Hyper-V role in the Windows Server operating system.
- **Linux operating system media.** Before you start this task, you must make sure that you have access to media that contains the Linux operating system. For a list of endorsed distributions, see [Linux on Windows Azure-Endorsed Distributions](../other-resources/linux-on-endorsed-distributions.md).

- **Linux Azure command-line tool.** If you are using a Linux operating system to create your image, you use this tool to upload the VHD file. To download the tool, see [Windows Azure Command-Line Tools for Linux and Mac](http://go.microsoft.com/fwlink/?LinkID=253691&clcid=0x409).
- **CSUpload command-line tool.** This tool is a part of the Windows Azure SDK. You use this tool to set the connection to Windows Azure and upload the VHD file. You must use the tools available in Windows Azure SDK - June 2012 or later to upload VHDs to Windows Azure. To download the SDK and the tools, see [Windows Azure Downloads](/en-us/develop/downloads/).

This task includes the following steps:

- [Step 1: Install the Hyper-V role on your server] []
- [Step 2: Create the image] []
- [Step 3: Create a storage account in Windows Azure] []
- [Step 4: Prepare the image to be uploaded] []
- [Step 5: Upload the image to Windows Azure] []

We also  have a generic section at the end with [Information for Non Endorsed Distributions][].

For all distributions note the following:

The Windows Azure Linux Agent (Waagent) is not compatible with NetworkManager. Networking configuration should use the ifcfg-eth0 file and should be controllable via the ifup/ifdown scripts. Waagent will refuse to install if the NetworkManager package is detected.

NUMA is not supported because the Linux kernel versions below 2.6.37 have a bug. The installation of waagent will automatically disable NUMA in the GRUB configuration for the Linux kernel command line.

The Windows Azure Linux Agent Requires python-pyasn1 package installed.

All of your VHDs for the OS must have sizes that are multiples of 1 MB.

## <a id="hyperv"> </a>Step 1: Install the Hyper-V role on your server ##

Multiple tools exist to create .vhd files. In this task, you use Hyper-V Manager to create the .vhd file that is uploaded to Windows Azure. For more information, see [Hyper-V](http://technet.microsoft.com/en-us/library/cc753637(WS.10).aspx).

1. On your server that is running Windows Server 2008, click **Start**, point to **Administrative Tools**, and then click **Server Manager**.

2. In the **Roles Summary** area, click **Add Roles**.

	![Add roles] (../media/role.png)

3. On the **Select Server Roles** page, click **Hyper-V**.

4. On the **Create Virtual Networks** page, click one or more network adapters if you want to make their network connection available to virtual machines.

5. On the **Confirm Installation Selections** page, click **Install**.

6. The computer must be restarted to complete the installation. Click **Close** to finish the wizard, and then click **Yes** to restart the computer.

7. After you restart the computer, log on with the same account you used to install the role. When the installation is complete, click **Close** to finish the wizard.

	You can now see the Hyper-V role installed on the server:

	![Hyper-V role added] (../media/rolehyperv.png)

## <a id="createimage"> </a>Step 2: Create the image ##

An image is a virtual hard disk (.vhd) file that you can use as a template to create a new virtual machine. An image is a template because it doesn’t have specific settings like a configured virtual machine, such as the computer name and user account settings. The .vhd file contains the operating system, any operating system customizations, and your applications. You can create the .vhd file by completing the following steps in Hyper-V.

1. On your server, click **Start**, click **All Programs**, click **Administrative Tools**, and then click **Hyper-V Manager**.

2. In the **Actions** pane of Hyper-V Manager, click **New**, and then click **Virtual Machine**.

	![Create virtual machine] (../media/newmachine.png)

3. In the New Virtual Machine Wizard, provide a name and a location for the virtual machine, the amount of memory that you want the virtual machine to use, and the network adapter that you want the virtual machine to use.

	You will be asked to provide information for the virtual hard disk that is used for creating the virtual machine.

	![Enter virtual machine details] (../media/newvhd.png)

4. On the Connect **Virtual Hard Disk** page, select **Create a virtual hard disk**. Provide the following information, and then click **Next**:

	- **Name** - the name of the .vhd file. This is the file that you upload to Windows Azure.
	- **Location** - the folder where the .vhd file is located. You should store the .vhd file in a secure location.
	- **Size** - the size of the virtual hard disk.  The maximum size for a virtual machine in Windows Azure is 127 GB.
5. On the **Installation Options** page, select **Install an operating system from a boot CD/DVD –ROM media**, and then choose the method that is appropriate for your installation media.

	![Choose the installation media] (../media/linuxchoosemedia.png)

6. Finish the wizard to create the virtual machine.

After the virtual machine is created it is not started by default. You must start the virtual machine to complete the installation of the operating system.

1. In the center pane of Hyper-V Manager, select the virtual machine that you created in the previous procedure.

2. In the **Actions** pane, click **Start**.

	![Start the virtual machine] (../media/start.png)

3. Click **Connect** to open the window for the virtual machine.

	![Connect to the virtual machine] (../media/connect.png)

4. Finish the installation of the operating system. For more information, see the documentation provided by the Linux distributor. You must also prepare the image by completing specific steps for the distribution that you are using. You do this in [Step 4: Prepare the image to be uploaded] []. 

	**Note:** It is recommended that you do not create a SWAP partition at installation time. You may configure SWAP space by using the Windows Azure Linux Agent. It is also not recommended to use the mainstream Linux kernel with a Windows Azure virtual machine without the patch available at the [Microsoft web site](http://go.microsoft.com/fwlink/?LinkID=253692&clcid=0x409).

## <a id="createstorage"> </a>Step 3: Create a storage account in Windows Azure ##

A storage account represents the highest level of the namespace for accessing the storage services and is associated with your Windows Azure subscription. You need a storage account in Windows Azure to upload a .vhd file to Windows Azure that can be used for creating a virtual machine. You can create a storage account by using the Windows Azure Management Portal.

1. Sign in to the Windows Azure Management Portal.

2. On the command bar, click **New**.

	![Create storage account] (../media/create.png)

3. Click **Storage Account**, and then click **Quick Create**.

	![Quick create a storage account] (../media/createnewstorage.png)

	The **Create a New Storage Account** dialog box appears.

	![Enter storage account details] (../media/storageinfo.png)

4. Enter a subdomain name to use in the URL for the storage account. The entry can contain from 3-24 lowercase letters and numbers. This value becomes the host name within the URL that is used to address Blob, Queue, or Table resources for the subscription.

5. Choose the region that will contain the storage account.

6. Choose whether you need geo-replication for the storage account. Geo-replication is turned on by default. During geo-replication, your data is replicated to a secondary region so that your storage fails over seamlessly to a secondary location in the event of a major failure that can't be handled in the primary location. The secondary location is assigned automatically, and can't be changed. If legal requirements or organizational policy requires tighter control over the location of your cloud-based storage, you can turn off geo-replication. However, be aware that if you later turn on geo-replication, you will be charged a one-time data transfer fee to replicate your existing data to the secondary location. Storage services without geo-replication are offered at a discount.

7. Click **Create Storage Account**.

	The account is now listed under **Storage Accounts**.

	![Storage account successfully created] (../media/storagesuccess.png)



## <a id="prepimage"> </a>Step 4: Prepare the image to be uploaded ##

### Prepare the CentOS 6.2 and CentOS 6.3 operating system ###

You must complete specific configuration steps in the operating system for the virtual machine to run in Windows Azure.

1. In the center pane of Hyper-V Manager, select the virtual machine.

2. Click **Connect** to open the window for the virtual machine.

3. Uninstall NetworkManager by running the following command:

		rpm -e --nodeps NetworkManager

	**Note:** If the package is not already installed, this command will fail with an error message. This is expected.

4.	Create a file named **network** in the `/etc/sysconfig/` directory that contains the following text:

		NETWORKING=yes
		HOSTNAME=localhost.localdomain

5.	Create a file named **ifcfg-eth0** in the `/etc/sysconfig/network-scripts/` directory that contains the following text:

		DEVICE=eth0
		ONBOOT=yes
		DHCP=yes
		BOOTPROTO=dhcp
		TYPE=Ethernet
		USERCTL=no
		PEERDNS=yes
		IPV6INIT=no

6. Enable the network service by running the following command:

		chkconfig network on

7. Install the drivers for the Linux Integration Services.
	
	a) Obtain the .iso file that contains the drivers for the Linux Integration Services from [Download Center](http://www.microsoft.com/en-us/download/details.aspx?id=34603).

	b) In Hyper-V Manager, in the **Actions** pane, click **Settings**.

	![Open Hyper-V settings] (../media/settings.png)

	c) In the **Hardware** pane, click **IDE Controller 1**.

	![Add DVD drive with install media] (../media/installiso.png)

	d) In the **IDE Controller** box, click **DVD Drive**, and then click **Add**.

	e) Select **Image file**, browse to **Linux IC v3.2.iso**, and then click **Open**.

	f) In the **Settings** page, click **OK**.

	g) Click **Connect** to open the window for the virtual machine.

	h) In the Command Prompt window, type the following commands:

		mount /dev/cdrom /media
		/media/install.sh`
		reboot

8. Install python-pyasn1 by running the following command:

		sudo yum install python-pyasn1

9. Replace their /etc/yum.repos.d/CentOS-Base.repo file with the following text
> [openlogic]
> name=CentOS-$releasever - openlogic packages for $basearch
> baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
> enabled=1
> gpgcheck=0
> 
> [base]
> name=CentOS-$releasever - Base
> baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
>  
> \#released updates
> [updates]
> name=CentOS-$releasever - Updates
> baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
>  
> \#additional packages that may be useful
> [extras]
> name=CentOS-$releasever - Extras
> baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
> gpgcheck=1
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
>  
> \#additional packages that extend functionality of existing packages
> [centosplus]
> name=CentOS-$releasever - Plus
> baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
> gpgcheck=1
> enabled=0
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
>  
> \#contrib - packages by Centos Users
> [contrib]
> name=CentOS-$releasever - Contrib
> baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
> gpgcheck=1
> enabled=0
> gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

10.	Add the following lines to /etc/yum.conf

-	http_caching=packages

-	exclude=kernel*
11. Disable the yum module "fastestmirror" by editing the file
"/etc/yum/pluginconf.d/fastestmirror.conf" and under [main]

-	set enabled=0  
12.	Clear the current Yum metadata
You need to clear your current yum metadata:
-	yum clean all


13. Update a Running VM's Kernel by running

-	For CentOS 6.2 do:
		-	sudo yum remove kernel-firmware
		-	sudo yum --disableexcludes=main install kernel-2.6.32-279.14.1.el6.openlogic.x86_64 kernel-firmware-2.6.32-279.14.1.el6.openlogic.x86_64

-	For CentOS 6.3 do:
		-		yum install kernel-2.6.32-279.14.1.el6.openlogic.x86_64.rpm


14. Ensure that you have modified the kernel boot line to include lines for 

-	console=ttyS0 ( this will enable serial console output)

-	rootdelay=300

15. Ensure that all SCSI devices mounted in your kernel include an I/O timeout of  300 seconds or more.

16.	Comment out Defaults targetpw in /etc/sudoers
17.	SSH Server should be included by default
18.	No SWAP on Host OS DISK should be created 
	SWAP if needed can be requested for creation on the local resource disk by the Linux Agent. You may modify /etc/waagent.conf appropriately.
19. Install the Windows Azure Linux Agent by running
-	yum install WALinuxAgent-1.2-1
20.	Run the following commands to deprovision the virtual machine:

		waagent –force –deprovision
		export HISTSIZE=0
		logout

21. Click **Shutdown** in Hyper-V Manager.

### Prepare the Ubunutu 12.04 and 12.10 operating system ###

1. In the center pane of Hyper-V Manager, select the virtual machine.

2. Click **Connect** to open the window for the virtual machine.

3.	Replace the current repositories in your image to use the azure repositories that carry the kernel and agent package that you will need to upgrade the VM.

	The steps for this are different depending on the distribution that you are using.

	Ubuntu 12.04 and 12.04.1:
	-	sudo sed -i “s,archive.ubuntu.com,azure.archive.ubuntu.com,g” /etc/apt/sources.list
	-	sudo apt-add-repository 'http://archive.canonical.com/ubuntu precise-backports main'
	-	sudo apt-get update

	Ubuntu 12.10:
	-	sudo sed -i “s,archive.ubuntu.com,azure.archive.ubuntu.com,g” /etc/apt/sources.list
	-	sudo apt-add-repository 'http://archive.canonical.com/ubuntu quantal-backports main'
	-	sudo apt-get update


4. Update the operating system to the latest kernel by running the following commands : 

		Ubuntu 12.04 and 12.04.1:
	-	sudo apt-get update
	-	sudo apt-get install hv-kvp-daemon-init linux-backports-modules-hv-precise-virtual
	-	(recommended) sudo apt-get dist-upgrade
	-	sudo reboot

	Ubuntu 12.10:
	-	sudo apt-get update
	-	sudo apt-get install hv-kvp-daemon-init linux-backports-modules-hv-quantal-virtual
	-	(recommended) sudo apt-get dist-upgrade
	-	sudo reboot


5.	 Install the agent by running the following commands with sudo:

		apt-get update
		apt-get install walinuxagent 

6.	Fix the Grub timeout. Ubuntu waits at Grub for user input after a system crash. To prevent this, complete the following steps:

	a) Open the /etc/grub.d/00_header file.

	b) In the function **make_timeout()**, search for **if [“\${recordfail}” = 1 ]; then**

	c) Change the statement below this line to **set timeout=5**.

	d) Run update-grub.

7. Ensure that you have modified the kernel boot line to include lines for 

-	console=ttyS0 ( this will enable serial console output)

-	rootdelay=300

8. It is recommended that you set /etc/sysconfig/network/dhcp or equivalent  from DHCLIENT_SET_HOSTNAME="yes" to DHCLIENT_SET_HOSTNAME="no"

9. Ensure that all SCSI devices mounted in your kernel include an I/O timeout of  300 seconds or more.
10.	Comment out Defaults targetpw in /etc/sudoers
11.	SSH Server should be included by default
12.	No SWAP on Host OS DISK should be created 
	SWAP if needed can be requested for creation on the local resource disk by the Linux Agent. You may modify /etc/waagent.conf appropriately.
	

13.	Run the following commands to deprovision the virtual machine:

		waagent –force –deprovision
		export HISTSIZE=0
		logout

14. Click **Shutdown** in Hyper-V Manager.

### Prepare the SUSE Linux Enterprise Server 11 SP2 operating system ###

1. In the center pane of Hyper-V Manager, select the virtual machine.

2. Click **Connect** to open the window for the virtual machine.

3. Addd the repository containing the latest agent and the latest kernel
  On the shell, call 'zypper lr'. If this command returns

	 | Alias                        | Name               | Enabled | Refresh
	
	1 | susecloud:SLES11-SP1-Pool    | SLES11-SP1-Pool    | No      | Yes    
	2 | susecloud:SLES11-SP1-Updates | SLES11-SP1-Updates | No      | Yes    
	3 | susecloud:SLES11-SP2-Core    | SLES11-SP2-Core    | No      | Yes    
	4 | susecloud:SLES11-SP2-Updates | SLES11-SP2-Updates | No      | Yes 

		then the repositories are configured as expected, no adjustments are necessary.

 	In case the command returns "No repositories defined. Use the 'zypper addrepo' command to add one or more repositories." then the repositories need to be re-enabled by calling 'suse_register -r'. You    		will get following output:

      Query installed languages failed.(134)  No packages found.
      To complete the registration, provide some additional parameters:

      Personal identification (mandatory) with:
        * E-mail address :    email="me@example.com"


      You can provide these parameters with the '-a' option.
      You can use the '-a' option multiple times.

      Example:

      suse_register -a email="me@example.com"

      To register your product manually, use the following URL:

      https://secure-www.novell.com/center/regsvc-1.0/?lang=POSIX&guid=64f60dc79688491e8bf88527804e06f0&command=interactive


      Information on Novell's Privacy Policy:
      Submit information to help you manage your registered systems.
      http://www.novell.com/company/policies/privacy/textonly.html

   Verify your repositories have been added by calling 'zypper lr', this is the expected output:

	    | Alias                        | Name               | Enabled | Refresh
     
      1 | susecloud:SLES11-SP1-Pool    | SLES11-SP1-Pool    | No      | Yes    
      2 | susecloud:SLES11-SP1-Updates | SLES11-SP1-Updates | No      | Yes    
      3 | susecloud:SLES11-SP2-Core    | SLES11-SP2-Core    | No      | Yes    
      4 | susecloud:SLES11-SP2-Updates | SLES11-SP2-Updates | No      | Yes 

   In case one of the relevant update repositories is not enabled, enable it with following command:

     # zypper mr -e [NUMBER OF REPOSITORY]

   In the above case, the command proper command would be

      # zypper mr -e 1 2 3 4

4. To get the ATA Piix driver, update the kernel to the latest available 
   version:

-	   \# zypper up kernel-default

5. Disable automatic DVD ROM probing.

6. Install the Windows Azure Linux Agent:

-	   Call 'zypper up WALinuxAgent' and you will get a similar message like the following:

      "There is an update candidate for 'WALinuxAgent', but it is from different vendor. 
      Use 'zypper install WALinuxAgent-1.2-1.1.noarch' to install this candidate."

   As the vendor of the package has changed from "Microsoft Corporation" to "SUSE LINUX Products GmbH, 
   Nuernberg, Germany", one has to explicitely install the package as mentioned in the message.

   Note: The version of the WALinuxAgent package might be slightly different.

7. To enable serial console and increase the rootdelay, adjust 
   the kernel command line in the file '/boot/grub/menu.lst' and add the 
   following string to the end of the kernel line:

   >console=ttyS0 rootdelay=300

8.	Ensure that all SCSI devices mounted in your kernel include an I/O timeout of  300 seconds or more (The SUSE agent intallation scripts normally take care of this).

9.	It is recommended that you set /etc/sysconfig/network/dhcp or equivalent  from DHCLIENT_SET_HOSTNAME="yes" to DHCLIENT_SET_HOSTNAME="no"

10.	Comment out Defaults targetpw in /etc/sudoers

11.	SSH Server should be included by default

12.	No SWAP on Host OS DISK should be created 
	SWAP if needed can be requested for creation on the local resource disk by the Linux Agent. You may modify /etc/waagent.conf appropriately.


13.	Run the following commands to deprovision the virtual machine:

		waagent –force –deprovision
		export HISTSIZE=0
		logout

14. Click **Shutdown** in Hyper-V Manager.

### Prepare the OpenSuse 12.1 operating system ###

1. In the center pane of Hyper-V Manager, select the virtual machine.

2. Click **Connect** to open the window for the virtual machine.

3. Update the operating system to the latest kernel.

	**Note:** The SLES kernel update does not currently contain an important fix on the kernel to improve storage performance. It is expected that this fix will be available soon after release. It is recommended that you use an image from the [SUSE Studio gallery]( http://www.susestudio.com) to take advantage of all the functionality in Windows Azure.

4. On the shell, call 'zypper lr'. If this command returns

         | Alias | Name | Enabled | Refresh
      
      1 | openSUSE_12.1_OSS     | openSUSE_12.1_OSS     | Yes     | Yes    
      2 | openSUSE_12.1_Updates | openSUSE_12.1_Updates | Yes     | Yes

   then the repositories are configured as expected, no adjustments are necessary.

   In case the command returns "No repositories defined. Use the 'zypper addrepo' command to add one 
   or more repositories." then the repositories need to be re-enabled:

      # zypper ar -f http://download.opensuse.org/distribution/12.1/repo/oss openSUSE_12.1_OSS
      # zypper ar -f http://download.opensuse.org/update/12.1 openSUSE_12.1_Updates

   Verify your repositories have been added by calling 'zypper lr', this is the expected output:

       | Alias | Name | Enabled | Refresh
     
      1 | openSUSE_12.1_OSS     | openSUSE_12.1_OSS     | Yes     | Yes    
      2 | openSUSE_12.1_Updates | openSUSE_12.1_Updates | Yes     | Yes

   In case one of the relevant update repositories is not enabled, enable it with following command:

      # zypper mr -e [NUMBER OF REPOSITORY]


5.	To get the ATA Piix driver, update the kernel to the latest available 
   version:

	   \# zypper in perl

   \# zypper up kernel-default

   Note: It is necessary to install the package 'perl' before updating the kernel, as there's
         a missing dependency in openSUSE 12.1.


7.	Disable automatic DVD ROM probing.

8.	Install the Windows Azure Linux Agent:

	First, add the repository containing the new WALinuxAgent:

      # zypper ar -f -r http://download.opensuse.org/repositories/Cloud:/Tools/openSUSE_12.1/Cloud:Tools.repo

   Then, call 'zypper up WALinuxAgent' and you will get a similar message like the following:

      "There is an update candidate for 'WALinuxAgent', but it is from different vendor. 
      Use 'zypper install WALinuxAgent-1.2-1.1.noarch' to install this candidate."

   As the vendor of the package has changed from "Microsoft Corporation" to "obs://build.opensuse.org/Cloud",
   one has to explicitely install the package as mentioned in the message.

   Note: The version of the WALinuxAgent package might be slightly different.

9.	To enable serial console and increase the rootdelay, adjust 
   the kernel command line in the file '/boot/grub/menu.lst' and add the 
   following string to the end of the kernel line:

   console=ttyS0 rootdelay=300

 	In the same file, remove the following part from the kernel command line:

      libata.atapi_enabled=0 reserve=0x1f0,0x8

10.	Ensure that all SCSI devices mounted in your kernel include an I/O timeout of  300 seconds or more (The SUSE agent intallation scripts normally take care of this).

11.	It is recommended that you set /etc/sysconfig/network/dhcp or equivalent  from DHCLIENT_SET_HOSTNAME="yes" to DHCLIENT_SET_HOSTNAME="no"

12.	Comment out Defaults targetpw in /etc/sudoers
13.	SSH Server should be included by default
14.	No SWAP on Host OS DISK should be created 
	SWAP if needed can be requested for creation on the local resource disk by the Linux Agent. You may modify /etc/waagent.conf appropriately.

15.	Run the following commands to deprovision the virtual machine:

		waagent –force –deprovision
		export HISTSIZE=0
		logout

16. Click **Shutdown** in Hyper-V Manager.

## <a id="upload"> </a>Step 5: Upload the image to Windows Azure ##

To upload an image contained in a VHD file to Windows Azure, you must:

1.	Create and install a management certificate
2.	Obtain the thumbprint of the certificate and the subscription ID
3.	Set the connection
4.	Upload the VHD file

### Create and install the management certificate ###

You need a management certificate uploaded to Windows Azure before you can upload a VHD.  For more information about creating certificates, see [Create Management Certificates for Linux](http://go.microsoft.com/fwlink/?LinkID=253693&clcid=0x409).

### Obtain the thumbprint of the certificate and the subscription ID ###

You need the thumbprint of the management certificate that you added and you need the subscription ID to be able to upload the .vhd file to Windows Azure.

1. From the Management Portal, click **Settings**.

2. Under **Management Certificates**, click your certificate, and then record the thumbprint from the **Properties** pane by copying and pasting it to a location where you can retrieve it later.

You also need the ID of your subscription to upload the .vhd file.

1. From the Management Portal, click **All Items**.

2. In the center pane, under **Subscription**, copy the subscription and paste it to a location where you can retrieve it later.

### Use the Linux command-line tool to upload the image ###

You can upload an image by using the following command:

		Azure vm image create <image name> --location <Location of the data center> --OS Linux <Sourcepath to the vhd>


### Use the CSUpload command-line tool to upload the image ###

You must set the connection string that is used to access the subscription. The CSUpload Command-Line Tool is used to set the connection string that is used. For more information, see [CSUpload Command-Line Tool](http://msdn.microsoft.com/en-us/library/gg466228.aspx).

1. Open a Windows Azure SDK Command Prompt window as an administrator.

2. Set the connection string by using the following command and replacing **Subscriptionid** and **CertThumbprint** with the values that you obtained earlier:

		csupload Set-Connection "SubscriptionID=<Subscriptionid>;CertificateThumbprint=<Thumbprint>;ServiceManagementEndpoint=https://management.core.windows.net"

After the connection string is set, you use the CSUpload command-line tool to upload a VHD file to the Image Gallery in Windows Azure.

1. Use the Windows Azure SDK Command Prompt window that you opened to set the connection string.

2. Set the connection string by using the following command and replacing **Subscriptionid** and **CertThumbprint** with the values that you obtained earlier:

		csupload Add-PersistentVMImage -Destination "<BlobStorageURL>/<YourImagesFolder>/<VHDName>" -Label <VHDName> -LiteralPath <PathToVHDFile> -OS Windows

	Where **BlobStorageURL** is the URL for the storage account that you created earlier. You can place the VHD file anywhere within your Blog storage. **YourImagesFolder** is the container within blob storage where you want to store your images. **VHDName** is the label that appears in the Management Portal to identify the VHD. **PathToVHDFile** is the full path and name of the VHD file.

## <a id="nonendorsed"> </a>Information for Non Endorsed Distributions ##
In essence all distributions running on Windows Azure will need to meet the following prerequisites to have a chance to properly run in the platform. 

This list is by no means comprehensive as every distribution is different; and it is quite possible that even if you meet all the criteria below you will still need to significantly modify your image to ensure that it runs properly in Windows Azure.

 It is for this reason that we recommend that you start with one of our [partners endorsed images](https://www.windowsazure.com/en-us/manage/linux/other-resources/endorsed-distributions/).

The list below replaces step 2 of the process to create your own VHD:

1.	You will need to ensure that you are running a kernel that either incorporates the latest LIS drivers for Hyper-V or that you have successfully compiled them ( They have been open sourced). The drivers can be found [at this location](http://go.microsoft.com/fwlink/p/?LinkID=254263&clcid=0x409).

2.	Your kernel should also include the latest version of the ATA PiiX driver that is used to to provision the iamges and has the fixes committed to the kernel with commit cd006086fa5d91414d8ff9ff2b78fbb593878e3c Date:   Fri May 4 22:15:11 2012 +0100   ata_piix: defer disks to the Hyper-V drivers by default.

3.	Your compressed intird should be less than 40 MB (* we are continuously working to increase this number so it might be outdated by now)

4.	You should add the following lines to your Kernel Boot

	-	console=ttyS0 rootdelay=300

5.	It is recommended that you set /etc/sysconfig/network/dhcp or equivalent  from DHCLIENT_SET_HOSTNAME="yes" to DHCLIENT_SET_HOSTNAME="no"

6.	You should ensure that all SCSI devices mounted in your kernel include an I/O timeout of  300 seconds or more.

7.	You will need to install the Agent following the steps in the [Agent Guide](https://www.windowsazure.com/en-us/manage/linux/how-to-guides/linux-agent-guide/). The Agent has been released under the Apache 2 license and you can get the latest bits at the [Agent GitHub Location](http://go.microsoft.com/fwlink/p/?LinkID=250998&clcid=0x409)
8.	Comment out Defaults targetpw in /etc/sudoers

9.	SSH Server should be included by default

10.	No SWAP on Host OS DISK should be created 
	SWAP if needed can be requested for creation on the local resource disk by the Linux Agent. You may modify /etc/waagent.conf appropriately.

11.	Run the following commands to deprovision the virtual machine:

        waagent –force –deprovision
        export HISTSIZE=0
        logout

12.	You will then need to shut down the virtual machine and proceed with the upload.


[Step 1: Install the Hyper-V role on your server]: #hyperv
[Step 2: Create the image]: #createimage
[Step 3: Create a storage account in Windows Azure]: #createstorage
[Step 4: Prepare the image to be uploaded]: #prepimage
[Step 5: Upload the image to Windows Azure]: #upload
[Information for Non Endorsed Distributions]: #nonendorsed