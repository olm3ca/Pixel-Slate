## How to build your own kernel for the Pixel Slate for audio support:
In the instructions for setting up audio in linux on the Pixel Slate, you can use the precompiled kernel available for download. Or, you can build your own.
These instructions are for Fedora systems, but can be easily adjusted to your distro of choice.

1. Get some software: 
`sudo dnf group install "C Development Tools and Libraries" "Development Tools"`
`sudo dnf install openssl`

1. Get kernel from Google:
`git clone --depth 1 https://chromium.googlesource.com/chromiumos/third_party/kernel -b chromeos-5.10`

1. To build, you'll need [this config developed by the Eupnea team](https://www.dropbox.com/s/f7tzfe1ta7hypzu/config?dl=0) - move it into the kernel folder created in the previous step, and rename it .config

1. Now it's time to build - this could take a while:

```
make olddefconfig
make -j$(nproc)   
sudo cp arch/x86/boot/bzImage /boot/vmlinuz-chromeos
sudo make modules_install INSTALL_MOD_STRIP=1
```
When this completes, look for a result from the DEPMOD line (for example g8330cd2908a1). Then run:

`sudo dracut /boot/initramfs-chromeos.img --kver 5.10.164-g8330cd2908a1`


### Set up grub to boot from this kernel

1. You'll need to add a grub entry:
`cd /boot/loader/entries/`

Look in this directory for the other boot entries. You'll want to copy one (not the rescue entry) and use it to build your new entry. 

`sudo touch chromeos.conf`

2. Based on the entries you found, add this to your chromeos.conf (modify it for the kernel name and root UUID)
```
title ChromeOS Kernel (5.10.164-g8330cd2908a1)
version 5.10.164-g8330cd2908a1
linux /vmlinuz-chromeos
initrd /initramfs-chromeos.img
options root=UUID=9f1e0a1d-3b40-49b9-a8a5-0b1aa0e12a38 ro  rhgb quiet
grub_users $grub_users
grub_arg --unrestricted
grub_class risios
```
Final step: update grub. Then reboot!

`sudo grub2-mkconfig -o /boot/grub2/grub.cfg`

