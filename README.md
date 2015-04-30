# Introduction
This document provides the steb-by-step instructions to configure your computer to use the binary distribution of the [LSST](http://www.lsst.org) software stack using [CernVM FS](http://cernvm.cern.ch/portal/filesystem).

CERN's CernVM FS is a software component which allow you to mount a remote software repository in **read-only mode**, which will appear to your computer as if the software was locally installed. At [CC-IN2P3](http://cc.in2p3.fr) we prepared a binary distribution of LSST stack to be used through CernVM FS. You will find below the procedure for installing CernVM FS and configuring it to use the binary LSST software repository.

Context and perspectives about this work can be found in [this presentation](https://speakerdeck.com/airnandez/experimenting-with-cernvm-fs-for-distributing-lsst-software).

**WARNING** : *please bear in mind that this work is experimental. Your feedback on how to improve it is very welcome. Scroll to the end of this document to know how you can provide feedback.*

# Expected Benefits
With this method, you need to install and configure CernVM FS only once. Once this is done, when your computer is connected to the network, you will find the available versions of the LSST software stack under the local directory:

	/cvmfs/lsst.in2p3.fr
	
This method of distributing the software is particularly useful for individuals: you don't need to install each official LSST software release by hand on your personal computer, but rather to automatically mount and use the binary distributions prepared for your convenience.

Please note that you don't need special privileges to *use* the LSST software stack distributed in this way: any user on a pre-configured computer can use the software. However, in order to *install* and configure CernVM FS, a one-time process, you need super-user privileges on the client machine.
	
# Installation
So far we have tested this installation on Scientific Linux 6, Scientific Linux 7, CentOS 7 and Ubuntu 14.04. It may work on other Linux platforms.

### Installing on Scientific Linux 6 and 7, CentOS 7 (64 bits)
To download the software from CERN's repository and install it, as `root` do:

   	# cd /tmp
	# curl -O https://ecsft.cern.ch/dist/cvmfs/cvmfs-2.1.19/cvmfs-2.1.19-1.el6.x86_64.rpm
	# curl -O https://cvmrepo.web.cern.ch/cvmrepo/yum/cvmfs/EL/5/x86_64/cvmfs-keys-1.5-1.noarch.rpm
	# yum install --assumeyes ./cvmfs-*.rpm

### Installing on Ubuntu 14.04 (64 bits)
Some packages are either necessary or recommended on Ubuntu. To install them do (as `root`):

    # apt-get install autofs attr gdb git sysv-rc-conf
    
Download and install CernVM FS (as `root`):
    
    # cd /tmp
    # curl -O https://ecsft.cern.ch/dist/cvmfs/cvmfs-2.1.19/cvmfs_2.1.19_amd64.deb
    # curl -O https://ecsft.cern.ch/dist/cvmfs/cvmfs-keys/cvmfs-keys_1.5-1_all.deb
    # dpkg -i ./cvmfs-keys_1.5-1_all.deb  ./cvmfs_2.1.19_amd64.deb
    

# Configuration
The configuration of CernVM FS client to use the binary distribution of LSST software served by CC-IN2P3 is a *one-time operation*. It needs to be performed by user `root`.

* Clone this repository and run the provided configuration script. The configuration script needs super-user privileges for creating or modifying some configuration files under `/etc/cvmfs`. You must run it as `root`:

		$ cd /tmp
		$ git clone https://github.com/airnandez/lsst-cvmfs.git
		$ cd lsst-cvmfs
		$ sudo sh ./configure.sh

	After this step, among other things, an unprivileged user `cvmfs` is created in your computer and several configuration files with sensible default values are located under `/etc/cvmfs`.
	
  You can tell the configuration process was successful if you won't see any error message.
		
* **[Recommended]** The CernVM FS client uses `autofs` for atuomatically mounting and unmounting the file system exposing the LSST software repository when needed. We suggest you to configure `autofs` to start at boot time. On Scientific Linux and CentOS do (as `root`):
	
		# chkconfig autofs on
		
	and on Ubuntu, do (as `root`):
	
		# sysv-rc-conf autofs on
		
Now you are ready to use the stack. See next section.
   		
# Usage
In order to use the LSST software stack, you need to setup your environment for a specific version for which there is a binary distribution available. For instance, to use LSST v9.2 do:

		$ cd /cvmfs/lsst.in2p3.fr/software/linux-x86_64/lsst-v9.2
		$ source loadLSST.sh
		
Note that you don't need super-user privileges to use this distribution of the LSST software. For testing your installation you can [run the LSST demo](https://confluence.lsstcorp.org/display/LSWUG/Testing+the+Installation).

# Available releases
At any moment, you can see what released are available for Linux-based machines by visiting the directory:

	/cvmfs/lsst.in2p3.fr/software/linux-x86_64
	
The releases for MacOS X will be availabe under:

	/cvmfs/lsst.in2p3.fr/software/darwin-x86_64
	
Currently you will find the releases presented in the table below:

| Platform                | Available versions of LSST software |
| ---------------------   |  ------------ |
| Linux, x86_64, 64bits   |  v9.2, v10.1-rc3 |
| Darwin, x86_64, 64bits  | v10.1-rc3 *(in preparation)* |


# Advanced usage
Details on how to use this distribution mechanism for more advanced use cases are provided in the [Advanced Usage](AdvancedUsage.md) document. There you will find details on how you can develop your own software package which depends on other packages already present in the binary distribution.


# Troubleshooting
Please note that in order for this distribution mechanism to work for you, you need your machine to be connected to the network and able to contact CC-IN2P3 server. To check this is the case please do:

	$ curl --proxy http://cctbcrnvmfsli01.in2p3.fr:3128 --head http://cccrnvmfs01.in2p3.fr/cvmfs/lsst.in2p3.fr/.cvmfspublished
	
You should see a line containning `HTTP/1.0 200 OK` which indicates that your machine can talk to the relevant server.

# Frequently Asked Questions

* **How can I provide feedback?**

  Your feedback is very welcome. Please feel free to [open an issue](https://github.com/airnandez/lsst-cvmfs/issues).

*  **Where can I get more detailed information on CernVM FS?**

	The [CernVM FS downloads page](http://cernvm.cern.ch/portal/filesystem/downloads) contains additional information. In addition, you may want to read the [CernVM FS Technical Information](http://cernvm.cern.ch/portal/filesystem/techinformation) for more in-depth information on how CernVM FS works.	

* **Can I use my remote LSST software distribution while disconnected from the network?**

  The CernVM FS client caches all the file metadata and the contents of the accessed remote files in the local disk of your computer. If you have previously used the stack it is likely that the relevant files are locally available in your local disk, in which case, you may work while disconnected. However, we have not tested this thoroughly, so let us know how it works for you.
  
* **Can I use this for my Docker containers?**

  Yes, you can configure your container for automatically mounting a read-only file system with LSST software stack ready to use. Sébastien Binet did exactly this, so you can just use as is or as a baseline for your own containers. You will find all the details [here](https://github.com/hepsw/docks/tree/master/cvmfs-lsst).

# Credits
This work was done by Fabio Hernandez from [IN2P3/CNRS computing center](http://cc.inp3.fr) (Lyon, France) with very valuable help from Vanessa Hamar who set up the CernVM FS server and proxy infrastructure.
