# ubuntu_t2mac

This guide allows for the installation of Ubuntu 20.04 on a MacbookPro with T2 chip. This works on my MacbookPro 2018.

## Installation

1. Go to the [repo](https://github.com/marcosfad/mbp-ubuntu) to obtain the lastest stable ISO of Ubuntu 20.04 image with T2 patches built-in. This allows you to install ubuntu with all appropriate drivers except WIFI.
2. Reduce the size of the mac partition in MacOS. Then, add two partitions: one for the Ubuntu operation system (to be mounted as /), and the other for the boot (1GB, to be mounted as /boot). Format these two partitions as MS-DOS.
3. Flash the ISO on to a bootable USB drive.
5. Boot in Recovery mode and allow booting unknown OS on the T2 chip
6. Restart and immediately press the option key until the Logo come up
7. Select "EFI Boot" (the third option was the one that worked for me)
8. Launch Ubuntu Installer
9. Select the options that work for you and use for the partition the following setup:
    * Add a ext4 partition and mounted as /boot (1024MB)
    * Add a ext4 partition and mounted as / (rest)
    * Set the bootload installation to the partition mounted as /
10. Continue and run the entire installer
11. Shutdown and remove the USB Drive
12. Start again using the option key. Select the new efi boot.
13. Enjoy.

## WIFI Configuration

Right now, all hardware should be working except for wifi. To obtain the correct wifi driver, see the following steps:
1. On MacOS, run `collect.sh` to obtain the bluetooth driver files which are stored in the `./driver-files` directory
2. These driver files are NOT compatible with Ubuntu. Go [here](https://packages.aunali1.com/apple/wifi-fw/18G2022/C-4364__s-B2/) to obtain the compatible ones with the same name.
3. Save the compatible files in the the `./downloaded-driver-files` directory
4. Reboot into Ubuntu. Run the script `install.sh`.
5. Verify that the driver files are installed in the correct place, i.e.,
```
# ls -l /lib/firmware/brcm | grep 4364
-rw-r--r--. 1 root root   12860 Mar  1 12:44 brcmfmac4364-pcie.Apple Inc.-MacBookPro15,1.txt
-rw-r--r--. 1 root root  922647 Mar  1 12:44 brcmfmac4364-pcie.bin
-rw-r--r--. 1 root root   33226 Mar  1 12:44 brcmfmac4364-pcie.clm_blob
```
6. Dynamically reload the driver
```
sudo modprobe -r brcmfmac
sudo modprobe brcmfmac
```
7. Verify if the driver is installed correctly. You should observe the following:
```
# dmesg | grep brcmfmac
brcmfmac 0000:01:00.0: enabling device (0000 -> 0002)
brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac4364-pcie for chip BCM4364/3
brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac4364-pcie for chip BCM4364/3
brcmfmac: brcmf_c_preinit_dcmds: Firmware: BCM4364/3 wl0: Mar 28 2019 19:17:52 version 9.137.9.0.32.6.34 FWID 01-36f56c94
brcmfmac 0000:01:00.0 wlp1s0: renamed from wlan0
```
Note: If you encounter a `Direct firmware failed to load` message, go to `/lib/firmware/brcm` and rename the files accordingly.

8. Install IWD and configure it:
```
sudo apt-get update && sudo apt-get install iwd
```
9. Ensure that `etc/NetworkManager/NetworkManager.conf` looks as follows:
```
[main]
plugins=ifupdown,keyfile

[ifupdown]
managed=false

[device]
wifi.scan-rand-mac-address=no
wifi.backend=iwd
```
10. Connect to WIFI and enjoy!

Note: Wifi sometimes drop connections when transferring large files. If wifi's connection kept dropping, reboot into MacOS then reboot back into Ubuntu to resolve this problem.

## For more information, visit:
- Ubuntu Installation Vlog: <https://vinodhsblog.co.za/ubuntu-20-04-lts-on-a-macbook-pro/>
- Ubuntu MacOS Github: <https://github.com/marcosfad/mbp-ubuntu>
- MacOS WIFI Drivers: <https://packages.aunali1.com/apple/wifi-fw/18G2022/>
- Current state of Ubuntu on MacOS: <https://github.com/Dunedan/mbp-2016-linux>


