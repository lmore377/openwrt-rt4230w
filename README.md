# openwrt-rac2v1k
Work-in-progress OpenWRT firmware for the RAC2V1K router from Askey

Follow progress here: https://forum.openwrt.org/t/askey-rac2v1k-support/15830

Steps to install

    Open the router and connect to the serial console
        Top lid is clipped on and taking it out reveals 2 screws
        If you see serial output but can't interrupt uboot, go look at this post: https://forum.openwrt.org/t/askey-rac2v1k-support/15830/21 You'll probably need to remove the wifi card to unscrew and remove the board.

    Install and set up a tftp server

    Set a static ip on the ethernet interface of your computer

    Download the factory image, rename it to factory.bin, and host it with the tftp server

    Connect the wan port of the router to your computer

    Interrupt U-Boot and run these commands:
    setenv serverip 10.42.0.1 (You can use whatever ip you set for the computer)
    setenv ipaddr 10.42.0.10 (Can be any ip as long as it's in the same subnet)
    tftpboot factory.bin

    After running tftpboot, the last line should look something like this: Bytes transferred = 6684672 (660000 hex). Make note of the hex number then run the following commands:
    nand erase 0x2400000 0x1A000000
    nand write 0x4400000 0x2400000 0x(hex number from tftpboot)
    setenv bootcmd "setenv mtdids nand0=nand0 && set mtdparts mtdparts=nand0:0x1A000000@0x2400000(firmware) && ubi part firmware && ubi read 0x44000000 kernel 0x6e0000 && bootm"
    saveenv
    reset
    -The router should boot to openwrt at this point
