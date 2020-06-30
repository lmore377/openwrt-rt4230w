# openwrt-rt4230w-rev6
Work-in-progress OpenWRT firmware for the RT4230W router from Askey

Follow progress here: https://forum.openwrt.org/t/askey-rac2v1k-support/15830

Warning: Do not flash OpenWRT on this router if it's currently being rented from Spectrum. The stock FW has a dual partition layout for redundancy and flashing this image will wipe out both partitions and a partition with info for provisioning with Spectrum, so there will be no way to revert it back to stock. I tried backing up/restoring the original parts but it caused the ubi partition for overlay to become corrupted so settings would be cleared after a reboot.

Note: Spectrum has a revision of this router that has no local web interface, a QR code on the back with SAC2V1K next to it, and the only way to change settings is through the My Spectrum app. This revision has a 256MB NAND chip but is otherwise identical. This image should work perfectly fine with that router but it's only been tested with initramfs so flash at your own risk.

Steps to install

    Open the router and connect to the serial console
        Top lid is clipped on and taking it out reveals 2 screws
        If you see serial output but can't interrupt uboot, go look at this post: https://forum.openwrt.org/t/askey-rac2v1k-support/15830/21 You'll probably need to remove the wifi card to unscrew and remove the board.

    Install and set up a tftp server

    Set a static ip on the ethernet interface of your computer

    Download the initramfs image, rename it to initramfs.bin, and host it with the tftp server

    Connect the wan port of the router to your computer

    Interrupt U-Boot and run these commands:
    setenv serverip 10.42.0.1 (You can use whatever ip you set for the computer)
    setenv ipaddr 10.42.0.10 (Can be any ip as long as it's in the same subnet)
    setenv bootcmd "setenv mtdids nand0=nand0 && set mtdparts mtdparts=nand0:0x1A000000@0x2400000(firmware) && ubi part firmware && ubi read 0x44000000 kernel 0x6e0000 && bootm"
    
    If you have a SAC2V1K router, use this bootcmd instead: "setenv mtdids nand0=nand0 && set mtdparts mtdparts=nand0:0xDC00000@0x2400000(firmware) && ubi part firmware && ubi read 0x44000000 kernel 0x6e0000 && bootm"
    
    saveenv
    tftpboot initramfs.bin
    bootm
    
    After openwrt boots, copy the sysupgrade file to a usb flash drive, plug it into the router, and run these commands:
    mount /dev/sda1 /mnt 
    cd /mnt 
    sysupgrade -F openwrt-ipq806x-generic-askey_rt4230w-squashfs-nand-sysupgrade.bin
    
    The router will reboot and if all went well, you'll now have openwrt running.
