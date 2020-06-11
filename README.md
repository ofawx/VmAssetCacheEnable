# VmAssetCacheEnable
Kernel patching method to allow enabling AssetCache (Content Caching) on macOS High Sierra and above, when running in a Virtual Machine.

## Introduction
Since macOS High Sierra, Apple prevents AssetCache being used on a virtualised machine.
On an *unpatched* system, executing `$ sysctl -a | grep cpu.features` will list the `VMM` flag as a CPU feature.
When AssetCache sees the `VMM` flag at startup, it prevents itself from running.

## Solution
These kernel patches modify the behaviour of sysctl, by replacing the `VMM` string with `XXX`.
On a *patched* system, executing `$ sysctl -a | grep cpu.features` will list an `XXX` flag *instead* of `VMM` as a CPU feature.
Thus, AssetCache runs normally, as if on a real Mac.

## Install
Apply the patch from the relevant .plist file to your Clover or OpenCore configuration.
[Re]Boot to allow patch to take effect.
Content Caching should now be available.

## Contrib
Feel free to raise any issues or PRs. Can't guarantee support but happy to try and help.
