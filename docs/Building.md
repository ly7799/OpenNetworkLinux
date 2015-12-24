#How to Build Open Network Linux 

In case you are not interested in building ONL from scratch
(it takes a while) you can download pre-compiled binaries from
http://opennetlinux.org/binaries .


Build Hosts and Environments
------------------------------------------------------------
ONL builds with Docker so the only requirements on the build system is:

- docker			# to grab the build workspace
- binfmt-support		# kernel support for ppc builds
- About 40G of disk free space 	# to build all images

All of the testing is done with Debian, other Linux distributions may work, but we suggest using Debian 8.
    # apt-get install lxc-docker binfmt-support


Build ONL Summary
------------------------------------------------------------
The easiest way to build is to use the autobuild script:

    #> git clone https://github.com/opencomputeproject/OpenNetworkLinux
    #> tools/autobuild/build.sh

This will build a Debian 7 based ONL from the master branch

To build a Debian 8 based ONL simply run:

    #> tools/autobuild/build.sh -8

If you would like to build by hand you can do the following:

    #> git clone https://github.com/opencomputeproject/OpenNetworkLinux
    #> cd OpenNetworkLinux
    #> make docker                                              # enter the docker workspace
    #> make amd64 ppc                           # make onl for $platform (currently amd64 or powerpc)

The resulting ONIE installers are in
$ONL/RELEASE/$SUITE/$ARCH/ONL-2.*INSTALLER, i.e. 
RELEASE/jessie/amd64/ONL-2.0.0_ONL-OS_2015-12-12.0252-ffce159_AMD64_INSTALLER
and the SWI files (if you want them) are in
$ONL/RELEASE/$SUITE/$ARCH/ONL*.swi. i.e.
RELEASE/jessie/amd64/ONL-2.0.0_ONL-OS_2015-12-12.0252-ffce159_AMD64.swi



#Installing Docker Gotchas

Docker installer oneliner (for reference: see docker.com for details)

    # apt-get install -y lxc-docker
or

    # wget -qO- https://get.docker.com/ | sh


Common docker related issues:

- Check out http://docs.docker.com/installation/debian/ for detailed instructions
- You may have to update your kernel to 3.10+
- Beware that `apt-get install docker` installs a dock application not docker :-)  You want the lxc-docker package instead.
- Some versions of docker are unhappy if you use a local DNS caching resolver:
	- e.g., you have 127.0.0.1 in your /etc/resolv.conf
        - if you have this, specify DNS="--dns 8.8.8.8" when you enter the docker environment
 	- e.g., `make DNS="--dns 8.8.8.8" docker`

Consider enabling builds for non-privileged users with:

- `sudo usermod -aG docker $USER`
- If you run as non-root without this, you will get errors like "..: dial unix /var/run/docker.sock: permission denied"	
- Building as root is fine as well (it immediately jumps into a root build shell), so this optional
    
#Additional Build Details
----------------------------------------------------------

The rest of this guide talks about how to build specific 
sub-components of the ONL ecosystem and tries to overview
all of the various elements of the build.

Build all .deb packages for all architectures
----------------------------------------------------------
    #> cd $ONL/packages
    #> make
    #> find $ONL/REPO -name \*.deb    # all of the .deb files end up here

A number of things will happen automatically, including:

- git submodule checkouts and updates for kernel, loader, and additional code repositories
- automatic builds of all debian packages and their dependencies
- automatic download of binary-only .deb packages from apt.opennetlinux.org

After all components have been built, your can build an ONL
Software Image from those components.

Adding/Removing packages from a SWI:
------------------------------------------------------------

The list of packages for a given SWI are in

    $ONL/packages/base/any/rootfs/common/$ARCH-packages.yml # for $ARCH specific packages
    $ONL/packages/base/any/rootfs/common/common-packages.yml	# for $ARCH-independent packages

Build a software image (SWI) for all powerpc platforms:
------------------------------------------------------------
    #> cd $ONL/builds/powerpc/swi
    #> make
    #> cd builds
    #> ls *.swi
    ONL-2.0.0_ONL-OS_2015-12-12.0252-ffce159_PPC.swi
    #>

Build an ONIE-compatible installer for all powerpc platforms.
This will incorporate the SWI you just built or build it dynamically if not.

This installer image can be served to ONIE on Quanta or Accton platforms:
------------------------------------------------------------
    #> cd $ONL/builds/powerpc/installer/legacy
    #> make
    #> cd builds
    #> ls *INSTALLER
    ONL-2.0.0_ONL-OS_2015-12-12.0252-ffce159_PPC_INSTALLER
    #>