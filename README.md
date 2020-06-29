# openwrt-rt4230w-rev6
Work-in-progress OpenWRT firmware for the RT4230W router from Askey

Follow progress here: https://forum.openwrt.org/t/askey-rac2v1k-support/15830

Warning: Don't install this if your renting the router from Spectrum. Since this image uses the space taken by both rootfs partitions, it'll be impossible to revert to stock and have it be stable (It just enters a bootloop and /overlay breaks)

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
    saveenv
    tftpboot initramfs.bin
    bootm
    
    After openwrt boots, copy the sysupgrade file to a usb flash drive, plug it into the router, and run these commands:
    mount /dev/sda1 /mnt 
    cd /mnt 
    sysupgrade -F openwrt-ipq806x-generic-askey_rt4230w-squashfs-nand-sysupgrade.bin
    
    The router will reboot and if all went well, you'll now have openwrt running.
