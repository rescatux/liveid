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
