# VmAssetCacheEnable
Kernel patching method to allow enabling AssetCache (Content Caching) on macOS High Sierra and above, when running in a Virtual Machine.

## Introduction
Since macOS High Sierra, Apple [*"explicitly disallows"*](https://support.apple.com/en-gb/HT207828) AssetCache being used on a virtualised machine.

In macOS High Sierra, Mojave and Catalina, on an *unpatched* VM system, executing:
* `$ sysctl -a | grep cpu.features` will list the `VMM` flag as a CPU feature

When AssetCache sees this value at startup, it prevents itself from running.

In macOS Big Sur and Monterey, on an *unpatched* VM system, executing:
* `$ sysctl -a | grep cpu.features` will list the `VMM` flag as a CPU feature, and
* `$ sysctl -a | grep kern.hv_vmm_present` will return `1` (True).

When AssetCache sees these values at startup, it prevents itself from running.

## Solution
These kernel patches modify the behaviour of sysctl, by replacing the `VMM` string with `XXX`.

On a *patched* system, executing `$ sysctl -a | grep cpu.features` will list an `XXX` flag *instead* of `VMM` as a CPU feature. Additionally, for macOS Big Sur and Monterey, the `kern.hv_vmm_present` field is renamed to `kern.hv_xxx_present`, so AssetCache cannot find it.

Thus, AssetCache runs normally, as if on a real Mac.

## Install

### Clover/OpenCore
 1. Apply the patch from the relevant .plist file to your Clover or OpenCore configuration.
 2. [Re]Boot to allow patch to take effect.
 3. Content Caching should now be available.

### Unraid Macinabox
**Note that this procedure has the potential to break your Macinabox install and/or VM. Please have a backup, and read through these instructions before attempting to ensure you are comfortable with the process *before* starting. If you require assistance, please consult the Macinabox support thread.**

Navigate to your Macinabox domain:

`$ cd /mnt/user/domains/MacinaboxCatalina`

Create a mount point for your Clover EFI image, enable NBD, attach the Clover image to an NBD device, and mount the EFI partition to the mount point:

```
$ mkdir efi_mount
$ modprobe nbd max_part=8
$ qemu-nbd --connect=/dev/nbd0 Clover.qcow2
$ mount /dev/nbd0p1 efi_mount/
```

You now want to edit `efi_mount/EFI/CLOVER/config.plist`. I strongly recommend making a backup first:

`$ cp efi_mount/EFI/CLOVER/config.plist config.plist.bkp`

If anything goes wrong from here, you can restore your backup:

```
$ rm efi_mount/EFI/CLOVER/config.plist
$ cp config.plist.bkp efi_mount/EFI/CLOVER/config.plist
```

Using a text or plist editor (I like [ProperTree](https://github.com/corpnewt/ProperTree)), into `efi_mount/EFI/CLOVER/config.plist`, copy the contents of `clover.config.plist`, not including the outermost `dict` and `plist` tags. You want to add `KernelAndKextPatches`, `KernelToPatch`, and patch element without damaging the rest of the file. Save and quit your editor.

Finally we can unmount the image:

`$ umount efi_mount`

To check you installed the patch okay, boot up your MacinaboxVM and in the Clover boot menu, go to `Options > Binaries Patching > Custom Kernel Patches` and ensure that `Block sysctl machdep.cpu.features VMM flag` is present and enabled. If Clover refuses to boot or this entry is not available, try installing the patch again, by mounting the image again and restoring your backup. Double-check you add in the patch correctly on your next attempt.

If all is going well, continue booting the VM, and verify that Content Caching can now be enabled.

Finally, disconnect the NBD device as we do not need it anymore:

```
$ qemu-nbd --disconnect /dev/nbd0
$ rmmod nbd
```

## Special notes
Model network type vmxnet3 is known to cause issues with Apple services.
It is strongly recommended to set network model type to e1000-82545em for macOS High Sierra, Mojave or Catalina, and virtio or virtio-net for macOS Big Sur and Monterey,

## Contrib
Feel free to raise any issues or PRs. Can't guarantee support but happy to try and help.
