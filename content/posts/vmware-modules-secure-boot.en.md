---
title: "VMware Modules and Secure Boot"
date: 2021-10-03T19:22:33+03:00
draft: false
tags: ["vmware", "secure-boot"]
---

## tl;dr

If you, like me, did not disable Secure Boot on your PC prior to installing Ubuntu, and if you, like me, would like to use [VMware Player](https://www.vmware.com/products/workstation-player.html)/Workstation (I haven't tested the Workstation, so ymmv) on your system, do the following:

1. launch VMware Player and get the "Before you can run VMware several modules must be compiled and loaded into the running kernel" window;
2. press the "build" button, wait for the build to seemingly fail (it will direct you to the build log);
3. observe that there's nothing terribly wrong with the build itself, just some warnings;
4. try loading the modules yourself (and get "Operation not permitted" error) - `dmesg` will have the messages along the lines of `Lockdown: modprobe: unsigned module loading is restricted; see man kernel_lockdown.7`;
5. navigate to `/var/lib/shim-signed/mok`, make sure there are two files there: `MOK.der` and `MOK.priv` (this is the important part, this is your Machine Owner Key on Ubuntu - it might be elsewhere on other systems);
6. perform signing with the machine owner key:

    ```bash
    $ sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 /var/lib/shim-signed/mok/MOK.priv /var/lib/shim-signed/mok/MOK.der $(modinfo -n vmmon)
    $ sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 /var/lib/shim-signed/mok/MOK.priv /var/lib/shim-signed/mok/MOK.der $(modinfo -n vmnet)
    ```

    (note that none of those commands will output anything);

7. run VMware Player again, to make sure it is working now;
8. reboot to have VMware add its network adapters (or fiddle with it yourself).

You have to have your kernel headers installed, but it's assumed you already have them (they're needed to build modules anyway).
