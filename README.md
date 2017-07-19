# Ubuntu 16.04 Unattended-Install for UEFI using preseed
Create a Legacy and UEFI automated Ubuntu ISO installer. Using preseed.cfg and post-install scripts, this can completely automate installing Ubuntu 16.04. This may work with other distros, but I have yet to test this. Your personal variables such as default packages, SSH keys and username/password will obviously need to be edited before use.

Follow the steps below to create your own image.

    cd /root
    wget http://releases.ubuntu.com/16.04.2/ubuntu-16.04.2-server-amd64.iso

### Extract ISO
    xorriso -osirrox on -indev ubuntu-16.04.2-server-amd64.iso -extract / custom-iso

### Edit /boot/grub/grub.cfg
This is for UEFI booting. This specifies the timeout on the grub menu to 1 second, tells grub what network adapter to use and grabs the preseed file.

    cd /root/custom-iso
    nano boot/grub/grub.cfg

Replace the contents with grub.cfg with the following. __Make sure you alter the HTTP location of your preseed file!__

    if loadfont /boot/grub/font.pf2 ; then
            set gfxmode=auto
            insmod efi_gop
            insmod efi_uga
            insmod gfxterm
            terminal_output gfxterm
    fi
    set default=0
    set timeout=1
    set menu_color_normal=white/black
    set menu_color_highlight=black/light-gray

    menuentry "Install Ubuntu Server" {
            set gfxpayload=keep
            linux /install/vmlinuz gfxpayload=800x600x16,800x600 hostname=ubuntu16-04 --- auto=true url=http://preseed.handsoff.local/ubuntu-16-04/preseed.cfg quiet
            initrd  /install/initrd.gz
    }

### Edit /isolinux/txt.cfg
If you do not want to enable preseeding on a legacy device, this step isn't necessary.

    nano isolinux/txt.cfg

Replace the contents with the following. __Make sure you alter the HTTP location of your preseed file!__

    GRUB_DEFAULT=0
    GRUB_HIDDEN_TIMEOUT=0
    GRUB_HIDDEN_TIMEOUT_QUIET=true
    GRUB_TIMEOUT=1

    default install
    label install
    menu label ^Install Ubuntu Server
    kernel /install/vmlinuz
    append preseed/url=http://preseed.handsoff.local/ubuntu-16-04/preseed.cfg bga=788 netcfg/choose_interface=auto initrd=/install/initrd.gz priority=critical --

## Obtain isohdpfx.bin for hybrid ISO

    cd ../
    sudo dd if=ubuntu-16.04.2-server-amd64.iso bs=512 count=1 of=custom-iso/isolinux/isohdpfx.bin

## Create new ISO

    cd custom-iso
    sudo xorriso -as mkisofs -isohybrid-mbr isolinux/isohdpfx.bin -c isolinux/boot.cat -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot -isohybrid-gpt-basdat -o ../custom-ubuntu-http.iso .
    
## Preseed file
Upload preseed .cfg file to location specified in grub.cfg. As we have specified networking within the Grub menu, we are now able to pull the preseed file over network.

You can either encrypt your password, or pass it over plain text. If you would like to hash the password, enter the following line. This will prompt for a password and return a hash for you to use. You must then add your hash to the preseed.cfg file.
    
    mkpasswd -m sha-512

## Still to do:
* Update Grub menu so can be used in Legacy mode
* Tidy preseed.cfg
* Confirm default packages to install
* Add post-install scripts
* Add snippet to create MD5 hashed password
* Add snippet to get post-install script, then execute
