# openwrt-rt4230w-rev6

## The pull request went through so I won't be building images anymore. You can find the snapshot builds [here.](https://downloads.openwrt.org/snapshots/targets/ipq806x/generic/)

Work-in-progress OpenWRT firmware for the RT4230W router from Askey (Branded as RAC2V1K and SAC2V1K by Spectrum)

Follow progress here: https://forum.openwrt.org/t/askey-rac2v1k-support/15830

Warning: Do not flash OpenWRT on this router if it's currently being rented from Spectrum. The stock FW has a dual partition layout for redundancy and flashing this image will wipe out both partitions and a partition with info for provisioning with Spectrum, so there will be no way to revert it back to stock. I tried backing up/restoring the original parts but it caused the ubi partition for overlay to become corrupted so settings would be cleared after a reboot.

Note: Spectrum has a revision of this router that has no local web interface, a QR code on the back with SAC2V1K next to it, and the only way to change settings is through the My Spectrum app. This revision usually has a 256MB NAND chip but is otherwise identical. This image should work perfectly fine with that router but it's only been tested with initramfs so flash at your own risk.

<details>
<summary>Method 1: Install without opening the case using SSH and tftp (Only works for RAC2V1K)</summary>
    
    Connect to one of the router's LAN ports
    
    Download the RAC2V1K-SSH.zip file and restore the config file that corresponds to your router's firmware (If you're firmware is newer than what's in the zip file, just restore the latest file)
    
    After a reboot, you should be able to ssh into the router with the username for your firmware in the readme.
    Run the following commannds:
    fw_setenv ipaddr 10.42.0.10 #IP of router, can be anything
    fw_setenv serverip 10.42.0.1# #IP of tftp server that's set up in next steps
    fw_setenv bootdelay 5
    fw_setenv bootcmd "tftpboot initramfs.bin; bootm; bootipq"
    
    Don't reboot the router yet.
    
    Install and set up a tftp server on your computer

    Set a static ip on the ethernet interface of your computer (use this for serverip in the above commands)

    Download the initramfs image, rename it to initramfs.bin, and host it with the tftp server
    
    Reboot the router. If you set up everything right, the router led should switch over to a slow blue glow which means openwrt is booted.
    After openwrt boots, ssh into it (root user, no password) and run these commands:
    fw_setenv bootcmd "setenv mtdids nand0=nand0 && set mtdparts mtdparts=nand0:0x1A000000@0x2400000(firmware) && ubi part firmware && ubi read 0x44000000 kernel 0x6e0000 && bootm"
    fw_setenv bootdelay 2
    
    After this, find some way to flash the sysupgrade image (luci, sftp, flash drive, etc.) 
    As the router reboots, unplug the ethernet cord to make sure it's not trying to boot over tftp again.
    The router will reboot and if all went well, you'll now have openwrt running.
</details>


<details>
<summary>Method 2: Install with serial access (Do this if something fails and you can't boot after using method 1) (Works with RAC2V1K and SAC2V1K) </summary>

    Open the router and connect to the serial console. Instructions can be found here: https://openwrt.org/inbox/toh/askey/rt4230w_rev6#opening_the_case

    Install and set up a tftp server

    Set a static ip on the ethernet interface of your computer

    Download the initramfs image, rename it to initramfs.bin, and host it with the tftp server

    Connect the wan port of the router to your computer

    Interrupt U-Boot and run these commands:
    setenv serverip 10.42.0.1 (You can use whatever ip you set for the computer)
    setenv ipaddr 10.42.0.10 (Can be any ip as long as it's in the same subnet)
    setenv bootcmd "setenv mtdids nand0=nand0 && set mtdparts mtdparts=nand0:0x1A000000@0x2400000(firmware) && ubi part firmware && ubi read 0x44000000 kernel 0x6e0000 && bootm"
    
    If you have a SAC2V1K router, use this bootcmd instead: 
    setenv bootcmd "setenv mtdids nand0=nand0 && set mtdparts mtdparts=nand0:0xDC00000@0x2400000(firmware) && ubi part firmware && ubi read 0x44000000 kernel 0x6e0000 && bootm"
    
    saveenv
    tftpboot initramfs.bin
    bootm
    
    After openwrt boots, figure out a way to flash the sysupgrade file (luci, sftp, flash drive, etc.)
    
    The router will reboot and if all went well, you'll now have openwrt running.
</details>

Thanks to: @efsg on the openwrt forums - Provided zip file for getting root
@eganov, @ghoffman, @efsg on the forum for help with testing
A few people on the irc channel for helping (Sorry, can't remember names.)
