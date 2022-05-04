# RHEL8-for-Davinci-Resolve18
RHEL8 for Davinci Resolve 18
# How to install DaVinci Resolve on RHEL8/CentOS 8.3

1. Create a bootable USB drive
	1. On Windows:
		1. Download [DVD ISO](https://www.centos.org/download/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. Download and use [Rufus](https://rufus.ie/) to create the bootable USB drive
	1. On Mac or Linux:
		1. Download [DVD ISO](https://www.centos.org/download/)
		1. [Verify the download](https://wiki.centos.org/TipsAndTricks/sha256sum)
		1. [Use `dd` to create the bootable USB drive](https://wiki.centos.org/HowTos/InstallFromUSBkey)
1. UEFI settings
	1. Set to boot to a USB drive first
	1. Disable Secure Boot and disable Legacy BIOS mode
1. Install CentOS from USB
	1. Software selection should be `Workstation` with only `GNOME Applications` checked.
	1. Set up DHCP
	1. Set password for `root` account and create just one administrator account
	1. Especially with a GeForce card, it's possible that the default CentOS installer will not show the graphics correctly. It might be necessary to install via the ["basic graphics mode" from the Troubleshooting menu in the boot options](https://docs.centos.org/en-US/centos/install-guide/Trouble-x86/#_problems_with_booting_into_the_graphical_installation).
1. CentOS's installation interacts with HP's UEFI in such a way as to change the boot order
	1. Reboot to the fresh installation of CentOS
	1. Accept the CentOS license
	1. You can then safely eject the USB installation disk
1. Install CentOS updates and reboot:
	
	```
	$ sudo dnf update --refresh
	$ sudo reboot
	```	
1. Install the [kernel source](https://wiki.centos.org/HowTos/I_need_the_Kernel_Source):
	
	```$ sudo dnf install "kernel-devel-uname-r == $(uname -r)"```
1. Install [EPEL](https://fedoraproject.org/wiki/EPEL)
	
	```$ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm```
1. Install DKMS
	
	```$ sudo dnf install dkms```
1. Install ELRepo
	1. Import the GPG key:
		
		```$ sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org```
		
	1. Install for CentOS 8:
	
		```$ sudo dnf install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm```

1. Installing EPEL should have downloaded and installed `gcc`, but just in case, make sure:

	```$ sudo dnf install gcc```

1. Install NVIDIA driver from ELRepo: 不建议使用ELREPO源安装nvidia闭源驱动。
建议使用rpmfusion源安装nvidia驱动。
1.2.添加rpmfusion，然后更新所有。命令：dnf install akmod-nvidia
禁用开源驱动
reboot,
dnf update
nvidia-smi

	1. Install the NVIDIA driver as well as the X11 driver:
	
		```$ sudo dnf install kmod-nvidia.x86_64 nvidia-x11-drv```
	
		The current version is `460.39-1.el8_3.elrepo`.
	
	1. [Configure Xorg as the default GNOME session](https://docs.fedoraproject.org/en-US/quick-docs/configuring-xorg-as-default-gnome-session/):
		1. `vim` with `sudo` permissions into `/etc/gdm/custom.conf`:
		
			```$ sudo vim /etc/gdm/custom.conf```
		
		1. Uncomment the line `WaylandEnable=false` by removing the `#` at the beginning of the line.
		
		1. In the same `[daemon]` section, add in the line: `DefaultSession=gnome-xorg.desktop`
		
		1. The `[daemon]` section of the file should now read:
		
			```
			[daemon]
			WaylandEnable=false
			DefaultSession=gnome-xorg.desktop
			```
			
	1. Then, reboot:
	
		```$ sudo reboot```
    
    或者手动在root用户登陆的情况下在桌面上安装nvidia.run.安装前完成系统所有更新，包括dkms，epel源。
    dnf install dkms,gcc
    重新启动系统
    
    禁用开源驱动。
    gedit /etc/gdm/custom.conf
    复制下面的内容
    [daemon]
			WaylandEnable=false
			DefaultSession=gnome-xorg.desktop
      
      重新启动系统
      
      登陆root账户
      给下载的nvidia.run文件，右键属性里勾选权限。
      终端里运行安装。
       。/NVIDIA-Linux------.run
       提示选择同意几次。结束后运行nvidia-smi。能正常运行，就属于驱动安装成功了。
    
    
    
1. [OPTIONAL] Download and install the latest DeckLink driver

	1. Download the latest driver [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/capture-and-playback)
	1. Become the `root` user:
		
		```$ su -```
		
		When prompted, enter your `root` user's password.
		
	1. If you already have an older DeckLink driver installed, uninstall it:
		
		```# rpm -qa | grep desktopvideo | xargs rpm -e```
		
	1. If GNOME didn't uncompress it for you already, uncompress the downloaded driver package:
		
		```# tar xvfz /path/to/downloaded/driver/location/Blackmagic_Desktop_Video_Linux_<driver_version>.tar.gz```
		
	1. `cd` into the `rpm` folder, since this is CentOS
	
		```# cd /Blackmagic_Desktop_Video_Linux_<driver_version>/rpm/<yourarchitecture>```
		
	1. Install the latest Desktop Video driver, GUI, and Media Express. Type:

		1. ```# dnf install desktopvideo-<driver_version>.x86_64.rpm```

		1. ```# dnf install desktopvideo-gui-<driver_version>.x86_64.rpm```
		
		1. ```# dnf install mediaexpress-<version>.x86_64.rpm```
		
			1. N.B. `dnf` should resolve dependencies and install `mesa-libGLU` for you.
		
	1. After the installation completes, you should see the terminal prompt. Reboot.
	1. After the machine has rebooted, open a Terminal shell again
	1. Become the `root` user again:
		
		```$ su -```
		
		When prompted, please enter your `root` user's password
		
	1. You might need to update the firmware on your DeckLink card. Type:
		
		```# BlackmagicFirmwareUpdater update 0```
		
	1.  If a firmware update was applied, reboot the machine after it completes. If no firmware update was required, a reboot is not necessary.

1. Now we should be totally ready for DaVinci Resolve.
	1. N.B. If you didn't already install `mesa-libGLU` for Media Express, Resolve definitely needs it, so make sure to install it:
		
		1. `$ sudo dnf install mesa-libGLU`
		
		1. Then, reboot.
		
1. [OPTIONAL] Install Java if you want to perform Photon validation of IMF packages:

	```$ sudo dnf install java```
	
	
1. [OPTIONAL] If you want to use your workstation as a PostgreSQL client for collaborative workflows, and the network is either air-gapped or has a trustworthy network-wide firewall, you'll want to disable the individual firewall on the workstation so that the east-west traffic between workstations will function properly: for bin locking, timeline locking, collaborative chat, etc.

	```
	$ sudo systemctl stop firewalld
	$ sudo systemctl disable firewalld
	```
		
1. Install DaVinci Resolve
	1. Download and extract `DaVinci_Resolve_Studio_16.2.8_Linux.zip` (if you have a DaVinci Resolve license dongle or key) or `DaVinci_Resolve_16.2.8_Linux.zip` [from the Blackmagic Design website](https://www.blackmagicdesign.com/support/family/davinci-resolve-and-fusion).
	1. Double-click the `.run` file to use the GUI installer
	1. Resolve might not launch after the installation--if you run it via the command-line from `/opt/resolve/bin/`, you can look for clues as to why it might not be able to launch. If some program is missing, try figuring out what Resolve needs and install via `dnf`.
