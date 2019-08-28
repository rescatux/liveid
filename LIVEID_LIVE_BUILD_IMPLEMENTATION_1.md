# liveid - live-build implementation 1

## Introduction
Liveid is a proposal or standard that needs to be followed by a couple of actors. So *distro maker tools* let you build your own distribution from a set of packages and configuration files. The usual output of such a tool is an ISO file that can be boot either as a CDROM or as an USB image.

Later on as a final user you might use a *Live CD remaster tool* in order to save many distros in a single usb device.

live-build is a *distro maker tool*. So this is what this implementation is going to be about. How to modify such a tool so that it [LIVEID proposal 1](LIVEID_PROPOSAL_1.md) is taken into account.

## Non official live-build disclaimer

This implementation if not from official live-build (the one from Debian) but from a live-build fork from Rescatux. So, unless I push these commits to upstream live-build in the future, please do not assume that these features are currently built-in in Debian's live-build.

## Live-boot modification
Although this page is titled liveid - live-build implementation live-build is not the only *distro maker tool* that needs to be modified. Live-boot also needs to be modified.
Live-boot is not strictly a *distro maker tool* but it's used by live-build in one of its steps towards building the final iso.
Live-boot builds an initramfs image which helps the kernel iso to find its own root device at boot. So, it's usually designed to find the big squashfs file and then boot from that new root system.

## live-build branch and commits

* Branch: [https://github.com/rescatux/live-build/tree/liveid-003](https://github.com/rescatux/live-build/tree/liveid-003)
* Commit: [Remove old unused UUID implementation](https://github.com/rescatux/live-build/commit/9c2814a8a34a1cb529e9376ae9ac9cccd9b00696)
* Commit: [Generate a public LB_UUID_FILE variable](https://github.com/rescatux/live-build/commit/3065ed25cea47311791fcd0ed54693307a46e5bd)
* Commit: [Set bootappend variables according to UUID](https://github.com/rescatux/live-build/commit/f97b06cb1cb20727ff3e8b8d9874b759aeafdef6)
* Commit: [Add UUID support to binary_grub-efi Secure Boot minimal grub.cfg](https://github.com/rescatux/live-build/commit/72ee23df27e4dffa2bfe46f9fc225736af9ded8a)

## live-boot branch and commits

* Branch: [https://github.com/rescatux/live-boot/tree/liveid-001](https://github.com/rescatux/live-boot/tree/liveid-001)
* Commit: [Added LIVEID parametre](https://github.com/rescatux/live-boot/commit/cf2d669808afe63dccd7928923b3d0c72b1aafc2)

## UUID generation

[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) should be generated at build time and it has to identify a given ISO.

Basically at build time, instead of generating a random UUID, you concatenate a series of build variables that make unique your CD.
Then you hash them so that you have something that changes a lot when a byte from your build variables have changed.

* Commit: [Generate a public LB_UUID_FILE variable](https://github.com/rescatux/live-build/commit/3065ed25cea47311791fcd0ed54693307a46e5bd)

You can see how many live-build configuration variables are concatenated here into the LB_UUID_SEED variable.

```
	# Setting LB_UUID_FILE (only once)
	if [ -z "${LB_UUID_FILE}" ]
	then
		LIVEID_DIR_PREFIX="LIVEID"

		LB_UUID_SEED="\
${APTITUDE_OPTIONS}\
-\
${APT_OPTIONS}\
-\
${_CONFFILE}\
-\
${_CONFIG}\
-\
${DEBOOTSTRAP_OPTIONS}\
-\
${DEBOOTSTRAP_SCRIPT}\
-\
${GZIP_OPTIONS}\
-\
${LB_APT}\
-\
${LB_APT_FTP_PROXY}\
-\
${LB_APT_HTTP_PROXY}\
-\
${LB_APT_INDICES}\
-\
${LB_APT_PIPELINE}\
-\
${LB_APT_RECOMMENDS}\
-\
${LB_APT_SECURE}\
-\
${LB_APT_SOURCE_ARCHIVES}\
-\
${LB_ARCHITECTURES}\
-\
${LB_ARCHIVE_AREAS}\
-\
${LB_BACKPORTS}\
-\
${LB_BINARY_FILESYSTEM}\
-\
${LB_BOOTAPPEND_INSTALL}\
-\
${LB_BOOTAPPEND_LIVE}\
-\
${LB_BOOTAPPEND_LIVE_FAILSAFE}\
-\
${LB_BOOTLOADERS}\
-\
${LB_BOOTSTRAP_QEMU_ARCHITECTURES}\
-\
${LB_BOOTSTRAP_QEMU_EXCLUDE}\
-\
${LB_BOOTSTRAP_QEMU_STATIC}\
-\
${LB_BUILD_WITH_CHROOT}\
-\
${LB_CACHE}\
-\
${LB_CACHE_INDICES}\
-\
${LB_CACHE_PACKAGES}\
-\
${LB_CACHE_STAGES}\
-\
${LB_CHECKSUMS}\
-\
${LB_CHROOT_FILESYSTEM}\
-\
${LB_COMPRESSION}\
-\
${LB_DEBCONF_FRONTEND}\
-\
${LB_DEBCONF_PRIORITY}\
-\
${LB_DEBIAN_INSTALLER}\
-\
${LB_DEBIAN_INSTALLER_DISTRIBUTION}\
-\
${LB_DEBIAN_INSTALLER_GUI}\
-\
${LB_DEBIAN_INSTALLER_PRESEEDFILE}\
-\
${LB_DISTRIBUTION}\
-\
${LB_FDISK}\
-\
${LB_FIRMWARE_BINARY}\
-\
${LB_FIRMWARE_CHROOT}\
-\
${LB_GRUB_SPLASH}\
-\
${LB_HDD_LABEL}\
-\
${LB_HDD_PARTITION_START}\
-\
${LB_HDD_SIZE}\
-\
${LB_INITRAMFS}\
-\
${LB_INITRAMFS_COMPRESSION}\
-\
${LB_INITSYSTEM}\
-\
${LB_INTERACTIVE}\
-\
${LB_ISO_APPLICATION}\
-\
${LB_ISOHYBRID_OPTIONS}\
-\
${LB_ISO_PREPARER}\
-\
${LB_ISO_PUBLISHER}\
-\
${LB_ISO_VOLUME}\
-\
${LB_JFFS2_ERASEBLOCK}\
-\
${LB_KEYRING_PACKAGES}\
-\
${LB_LINUX_FLAVOURS_WITH_ARCH}\
-\
${LB_LINUX_PACKAGES}\
-\
${LB_LOADLIN}\
-\
${LB_LOSETUP}\
-\
${LB_MEMTEST}\
-\
${LB_MIRROR_BINARY}\
-\
${LB_MIRROR_BINARY_SECURITY}\
-\
${LB_MIRROR_BOOTSTRAP}\
-\
${LB_MIRROR_CHROOT}\
-\
${LB_MIRROR_CHROOT_SECURITY}\
-\
${LB_MIRROR_DEBIAN_INSTALLER}\
-\
${LB_MODE}\
-\
${LB_NET_COW_FILESYSTEM}\
-\
${LB_NET_COW_MOUNTOPTIONS}\
-\
${LB_NET_COW_PATH}\
-\
${LB_NET_COW_SERVER}\
-\
${LB_NET_ROOT_FILESYSTEM}\
-\
${LB_NET_ROOT_MOUNTOPTIONS}\
-\
${LB_NET_ROOT_PATH}\
-\
${LB_NET_ROOT_SERVER}\
-\
${LB_NET_TARBALL}\
-\
${LB_ONIE}\
-\
${LB_ONIE_KERNEL_CMDLINE}\
-\
${LB_PARENT_ARCHIVE_AREAS}\
-\
${LB_PARENT_DEBIAN_INSTALLER_DISTRIBUTION}\
-\
${LB_PARENT_DISTRIBUTION}\
-\
${LB_PARENT_MIRROR_BINARY}\
-\
${LB_PARENT_MIRROR_BINARY_SECURITY}\
-\
${LB_PARENT_MIRROR_BOOTSTRAP}\
-\
${LB_PARENT_MIRROR_CHROOT}\
-\
${LB_PARENT_MIRROR_CHROOT_SECURITY}\
-\
${LB_PARENT_MIRROR_DEBIAN_INSTALLER}\
-\
${LB_SECURITY}\
-\
${LB_SOURCE}\
-\
${LB_SOURCE_IMAGES}\
-\
${LB_SWAP_FILE_PATH}\
-\
${LB_SWAP_FILE_SIZE}\
-\
${LB_SYSTEM}\
-\
${LB_TASKSEL}\
-\
${LB_UEFI_SECURE_BOOT}\
-\
${LB_UPDATES}\
-\
${LB_WIN32_LOADER}\
-\
${LB_ZSYNC}\
-\
${LIVE_IMAGE_NAME}\
-\
${LIVE_IMAGE_TYPE}\
"
```
This LB_UUID_SEED variable can be hashed thanks to md5sum into a nice uuid which it's saved into the LB_UUID variable.
```
		LB_UUID=$(echo -n "${LB_UUID_SEED}" | md5sum | tr 'a-z' 'A-Z')
```
Finally the UUID is rearranged so that it can be put into the media as a series of directories and a file on top of LIVEID/ folder.
```
		LB_UUID_DIR1="$(echo ${LB_UUID} | cut -c1-8)"
		LB_UUID_DIR2="$(echo ${LB_UUID} | cut -c9-16)"
		LB_UUID_DIR3="$(echo ${LB_UUID} | cut -c17-24)"
		LB_UUID_FILE4="$(echo ${LB_UUID} | cut -c25-32)"

		LB_UUID_DIR="${LIVEID_DIR_PREFIX}/${LB_UUID_DIR1}"'/'"${LB_UUID_DIR2}"'/'"${LB_UUID_DIR3}"
		LB_UUID_FILE="${LB_UUID_DIR}"'/'"${LB_UUID_FILE4}"
	fi
```


## Finding grub.cfg - Solution

* Commit: [Add UUID support to binary_grub-efi Secure Boot minimal grub.cfg](https://github.com/rescatux/live-build/commit/72ee23df27e4dffa2bfe46f9fc225736af9ded8a)

The minimal grub.cfg for Secure Boot can be generated at build time.

What it usually was:

```
mkdir -p ${_CHROOT_DIR}/grub-efi-temp-cfg
cat >${_CHROOT_DIR}/grub-efi-temp-cfg/grub.cfg <<EOF
search --set=root --file /live/vmlinuz
set prefix=(\\\$root)/boot/grub
configfile (\\\$root)/boot/grub/grub.cfg
EOF
```
gets converted into:
```
mkdir -p ${_CHROOT_DIR}/grub-efi-temp-cfg
cat >${_CHROOT_DIR}/grub-efi-temp-cfg/grub.cfg <<EOF
search --set=root --file /${LB_UUID_FILE}
set prefix=(\\\$root)/boot/grub
configfile (\\\$root)/boot/grub/grub.cfg
EOF
```
.

That way you no longer depend on a very common kernel name (vmlinuz) but on the very unique UUID for this image.

## Finding squashfs file - Solution
(You can do the same with isolinux boot entries.)

* Commit: [Set bootappend variables according to UUID](https://github.com/rescatux/live-build/commit/f97b06cb1cb20727ff3e8b8d9874b759aeafdef6)

Kernel has to receive as a boot parametre what's the media correct LIVEID.
We will use the *liveid* boot parametre for that.

We will just append the correct *liveid* string to the already existing variables that define the default kernel boot parametres when creating the iso.

```
# Setting bootapend according to UUID
LB_BOOTAPPEND_LIVE="${LB_BOOTAPPEND_LIVE} liveid=/${LB_UUID_FILE}"
LB_BOOTAPPEND_LIVE_FAILSAFE="${LB_BOOTAPPEND_LIVE_FAILSAFE} liveid=/${LB_UUID_FILE}"
LB_BOOTAPPEND_INSTALL="${LB_BOOTAPPEND_INSTALL} liveid=/${LB_UUID_FILE}"
```

## Extra features 1 - Not implemented

This might be useful for remaster tools in order to get the correct name of the distro and, also, to identify the type of it.

  * /LIVEID/VENDOR: Ubuntu
  * /LIVEID/NAME: Bionic Beaver
  * /LIVEID/VERSION: 18.04
  * /LIVEID/BUILDTOOL: Casper 1.394
  * /LIVEID/UUID: /LIVEID/123E4567/E89B12D3/A4564266/55440000


## Extra features 2 - Not implemented

This might be useful for remaster tools in order to get the correct name of the distro and, also, to identify the type of it.

* /LIVEID/LIVEID.CFG with the following contents:

```
liveid_vendor="Ubuntu"
liveid_name="Bionic Beaver"
liveid_version="18.04"
liveid_buildtool="Casper 1.394"
liveid_uuid="/LIVEID/123E4567/E89B12D3/A4564266/55440000"


export liveid_vendor
export liveid_name
export liveid_version
export liveid_buildtool
export liveid_uuid

```

