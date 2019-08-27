# liveid
GNU/Linux Live CD / USB ID (liveid) proposal

# What is liveid?

It's a proposal to try to make it easier to identify GNU/Linux live CDs (Nowadays are also available to be put in USB but we will call them live cds anyways).

# Why do we need liveid?

Live CDs should be able to detect themselves when booted either from a CD or an USB.

# Additional liveid benefits

CD Remaster tools to be able to improve their tools and avoid their generated live CDs to conflict (detect an unexpected live CD and try to boot into it).

# Original issue

Having found boot issues factory installed HP machines issues while trying a live-build generated iso (Rescatux) More information at : [live-build: UEFI shows grub> on HP250 G6 2SX60EA](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=924053) .

# Problem 1 - Finding grub.cfg

This is a recent problem that face Secure Boot enabled GNU/Linux live cds.
Debian implementation of Secure Boot in live-build involves using a cut version of grub where most of the modules have been removed.
That means, in practice, that this grub instance does not know how to find the live cd's grub.cfg file.

The [current implementation](https://salsa.debian.org/live-team/live-build/blob/bd7c900d3ef2e17c0092a430783a0bf25c740ec1/scripts/build/binary_grub-efi#L221-226):
```
search --set=root --file /live/vmlinuz
set prefix=($root)/boot/grub
configfile ($root)/boot/grub/grub.cfg
```

tries to identify where the /boot/grub/grub.cfg should be by searching on all of the grub devices for the /live/vmlinuz file.


At first glance this is a better approach than searching for /boot/grub/grub.cfg file. Why? Because if you already have an installed GNU/Linux operating system in your computer it's highly likely that it has grub2 as its bootloader with its usual /boot/grub/grub.cfg. So instead of searching for grub.cfg file which would have booted your internal hard disk GNU/Linux (instead of the external one) we search for /live/vmlinuz. Why? Because such a file like /live/vmlinuz is only used on live CDs.

Well, this is not actually true. /live/vmlinuz is used in factory-installed-HPs cause they use a live-build base system to show its documentation and warranty.
This file would be also be a problem if you happen to connect two GNU/Linux Live USBs at the same time and your firmware (BIOS/UEFI) shows both of them to grub.


My first [easy approach on solving this problem](https://github.com/rescatux/live-build/commit/8e5f36048559a00abbd7102ab0f7ecb2f278bf4f) was using /.disk/info file instead.


This might be a workaround for avoiding problems with factory-installed-HPs computers but it's postponing the big problem here.
When you install a distribution into an internal hard disk you usually use filesystem UUIDs for detecting that particular installation.
That's the way it should be done also in live cds/usbs.


# Problem 2 - Finding squashfs file

Space is not cheap. Most of GNU/Linux live cds use compressed read-only file systems which sometimes are mixed with ram so that they can be writable (not on your cdrom or usb but on ram). This lets you put, say, 1 GB into a 700 MB disk.

So basically if you open a live-build generated cdrom you will find this huge file named: /live/filesystem.squashfs .


I'm going to repeat myself again. When you install a distribution into an internal hard disk you usually use filesystem UUIDs for detecting that particular installation. So that means that your bootloader, usually grub2 knows how to find your installation kernel and your installation initrd.

```
menuentry mylinux
        insmod gzio
        insmod part_gpt
        insmod ext2
        set root='hd0,gpt5'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt5 --hint-efi=hd0,gpt5 --hint-baremetal=ahci0,gpt5  b51cdb6a-e9db-4e19-9f1e-fe7a7fcd7c87
        else
          search --no-floppy --fs-uuid --set=root b51cdb6a-e9db-4e19-9f1e-fe7a7fcd7c87
        fi
        echo    'Loading Linux 4.9.0-4-amd64...'
        linux   /boot/vmlinuz-4.9.0-4-amd64 root=UUID=b51cdb6a-e9db-4e19-9f1e-fe7a7fcd7c87 ro resume=/dev/sda7  video=efifb i915.lvds_channel_mode=2 i915.modeset=1 i915.lvds_use_ssc=0 log_buf_len=16M
        echo    'Loading ramdisk...'
        initrd  /boot/initrd.img-4.9.0-4-amd64
```


It boots into them passing the filesystem UUID as a kernel parametre and the kernel loads the initrd into memory (that's why it's an init ram disk). The initrd is the one which actually searches for the filesystem UUID.

Both the kernel and the initrd have the minimal components to be able to find that specific filesystem UUID, mount that filesystem UUID and continue with the usual GNU/Linux boot process where daemons and services begin to start and do something useful.

As I said that's a job mainly done by initrd.

In the case of live-build its initrd is being built thanks to live-boot package. So what's the filesystem UUID search equivalent on live-boot [1](https://salsa.debian.org/live-team/live-boot/blob/7130a2c0b0697df5985346dbb3bf2a1e2d5b142e/components/0001-init-vars.sh#L5) [2](https://salsa.debian.org/live-team/live-boot/blob/7130a2c0b0697df5985346dbb3bf2a1e2d5b142e/components/9990-misc-helpers.sh#L5-16)?

```
LIVE_MEDIA_PATH="live"
```


```
is_live_path()
{
	DIRECTORY="${1}/${LIVE_MEDIA_PATH}"
	for FILESYSTEM in squashfs ext2 ext3 ext4 xfs dir jffs
	do
		if ls "${DIRECTORY}/"*.${FILESYSTEM} > /dev/null 2>&1
		then
			return 0
		fi
	done
	return 1
}

```

Well, basically live-boot is searching for:

```
/live/*.squashfs
/live/*.ext2
/live/*.ext3
/live/*.ext4
/live/*.xfs
/live/*.dir
/live/*.jffs
```

which matches with:

```
/live/filesystem.squashfs
```
file.

And, guess what, there is such an squashfs file in some factory-installed-HPs computers which converts a normal GNU/Linux live cd boot into this prompt:
```
Progress Linux 1.9
```
.

# loopback.cfg solution

Before going into what would be the liveid technical proposal I would like to share loopback.cfg with CD remaster tools developers.

loopback.cfg is designed to solve two problems:
* No need to unpack the ISO to be able to boot from it.
* No need to figure out the different boot options. The distro gives them for you.

loopback.cfg asummes you are either using a bootloader capable of loop-mounting an iso9660 file (such grub2) or that previously unpacked only the kernel and its initrd.

You can find more information about loopback.cfg in [https://www.supergrubdisk.org/wiki/Loopback.cfg](https://www.supergrubdisk.org/wiki/Loopback.cfg).

I am sure the wiki page can be improved a lot so that the different users of it (Distro developer, remaster tool developer and final user) know what they are supposed to do with it.

Given an iso with loopback.cfg in it you should be able to boot into it thanks to:
```
menuentry "TITLE" {
  iso_path=PATH
  export iso_path
  search --set=root --file $iso_path
  loopback loop $iso_path
  root=(loop)
  configfile /boot/grub/loopback.cfg
  loopback --delete loop
}
```
. An example would be:
```
menuentry "TITLE" {
  iso_path=/myisos/rescatux-0.71b2.iso
  export iso_path
  search --set=root --file $iso_path
  loopback loop $iso_path
  root=(loop)
  configfile /boot/grub/loopback.cfg
  loopback --delete loop
}
```

The distro needs to support loopback.cfg and usually it supports a findiso= kernel parametre which it's used by its initrd.

The only problems about GNU/Linux cds not being identified correctly would happen if you had this same exact path and filename: /myisos/rescatux-0.71b2.iso in more than one partition. That's highly unlikely.


Anyway what I meant to say here is that if you are already using loopback.cfg then the liveid solution might not be needed at all. The iso file path and filename takes care of it for you.


# liveid - Restrictions

We already explained **Problem 1 - Finding grub.cfg** where grub2 needs to find the media where its grub.cfg is found. Using grub2 limits us a lot because they do not have a way of saving first line of a file into a variable.
```
MYLINE=$(head -n 1 /tmp/myfile.txt)
```
.
And even if they had you would have to find a way to search for this file and then try to read it and, then, if it its content does not match what you expected then you have to loop into another result of the search.

And that makes it awful to implement. Take a look at this hipotetical grub menu code:

```
search --set=uuidroot --file /live/uuid
headone --set=uuidcontent ($uuidroot)/live/uuid
if [ "$uuidcontent" == "123e4567-e89b-12d3-a456-426655440000" ] ; then
    set root=($uuidroot)
    set prefix=($root)/boot/grub
    configfile ($root)/boot/grub/grub.cfg
else
    echo "How do I search for the same file on next device?"
fi
```

So what we want is a name that we can compare with our UUID.

# live id - Proposal 1

## UUID generation
[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) should be generated at build time and it has to identify a given ISO.

Basically at build time, instead of generating a random UUID, you concatenate a series of build variables that make unique your CD.
Then you hash them so that you have something that changes a lot when a byte from your build variables have changed.
In the case of md5sum the result matches to an 32 hexadecimal result.

Why you don't want a random UUID? Because we want the build process to be reproducible.


We can just save the resultant UUID into the live cd as:
/LIVEID/123E4567/E89B12D3/A4564266/55440000
.
(Note the capital letters)

## Finding grub.cfg - Solution
The minimal grub.cfg for Secure Boot can be generated at build time:

```
search --set=root --file /LIVEID/123E4567/E89B12D3/A4564266/55440000
set prefix=($root)/boot/grub
configfile ($root)/boot/grub/grub.cfg
```

## Finding squashfs file - Solution
(You can do the same with isolinux boot entries.)

So, where you had before:
```
menuentry "Live system, kernel 4.19.0-2-amd64" {
    linux /live/vmlinuz-4.19.0-2-amd64 boot=live config quiet splash locales=en_US.UTF-8
    initrd /live/initrd.img-4.19.0-2-amd64
}
```
now you have:
```
menuentry "Live system, kernel 4.19.0-2-amd64" {
    linux /live/vmlinuz-4.19.0-2-amd64 liveid=/LIVEID/123E4567/E89B12D3/A4564266/55440000 boot=live config quiet splash locales=en_US.UTF-8
    initrd /live/initrd.img-4.19.0-2-amd64
}
```

## How to implement live id

  * Find a way to identify your iso and convert that string into an UUID
  * Modify your minimal Secure Boot image cfg generator to search for such an UUID
  * Modify your live's initrd to take into account the liveid parametre.

## Extra features

This might be useful for remaster tools in order to get the correct name of the distro and, also, to identify the type of it.

  * /LIVEID/VENDOR: Ubuntu
  * /LIVEID/NAME: Bionic Beaver
  * /LIVEID/VERSION: 18.04
  * /LIVEID/BUILDTOOL: Casper 1.394
  * /LIVEID/UUID: /LIVEID/123E4567/E89B12D3/A4564266/55440000

Note: I don't actually know what Ubuntu people are using to build their live cds. Several years ago I thought it was Casper but I might be wrong on that. Pull requests are welcomed to fix this.

# Discarded proposals

[Discarded proposals](DISCARDED_PROPOSALS.md)

# Feedback needed

I was going to implement this new way of identying live cds in live-build so that I could use in Rescatux.
Although I have contributed to live-build (and live-boot in the past) I don't consider myself as a developer of live cd build tools.
And, as made my own distro, Rescatux, I often Refuse to use remaster tools to put many distros into a usb.

I'm sure that many live cd remaster tools do not know about isolinux.cfg.
And I'm concerned about them not using the liveid parametre because of them wanting to put each one of the distro contents on several directories.

So I need feedback from both the people who develop tools to build live CDs as well as feedback from people who develop CD remaster tools.

For the live cd build tools developers I would ask them if they find this difficult to implement.

And for the remaster tools developers I would ask them, discarding the Secure Boot minimal grub, if cdrom isos with this new liveid proposal applied to them were to be found in the wild...
if they would have any problem on remastering them.


Thank you for your feedback!
