

# Grub Rescue
>[!NOTE]
>ls (hd0)(hd0,gpt1)(hd0,gpt2)
>admettons gpt2=sda2 et contient a la suite de (hd0,gpt2)/ les fichier de configuration

    ls
    ls (hd0,gpt2)/
    cat (hd0,gpt2)/etc/issue
    ls -lh (hd0,gpt2)/boot
    set prefix=(hd0,gpt2)/boot/grub
    set root=(hd0,gpt2)
    insmod normal
    insmod linux
    linux /boot/vmlinuz-6.2.0-20-generic root=/dev/sda5
    initrd /boot/initrd.img-6.2.0-20-generic
    boot

# Initramfs
    cat /proc/partitions 
    blkid
    lsblk
    set prefix
    fsck /dev/sda5 -y 
    reboot -f



# FSTAB
    sudo mkdir /media/distro
    sudo apt install arch-install-scripts
    genfstab -U -p /media/distro >> /media/distro/etc/fstab


# GRUB LEGACY
    sudo mkdir /media/distro
    lsblk
    sudo mount /dev/sda5 /media/distro
    sudo mount --bind /dev /media/distro/dev
    sudo mount -t proc /proc /media/distro/proc
    sudo mount -t sysfs /sys /media/distro/sys
    sudo chroot /media/distro
    grub-install /dev/sda
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    update-grub
    reboot
    sudo cp /usr/share/grub/defaults/ /etc/default/grub



# LEGACY to UEFI

> [!WARNING]  
>gparted creer un partition de 512MB fat32

    sudo apt install grub-efi
    cd /media
    sudo mkdir ROOT
    sudo mkdir EFI
    sudo mount /dev/sda2 /media/ROOT
    sudo mount /dev/sda1 /media/EFI
    sudo grub-install --target=x86_64-efi /dev/vda --efi-directory=/media/EFI --boot-directory=/media/ROOT/boot
    cd /media/ROOT/etc
    sudo apt install arch-install-scripts
    genfstab -U -p /media/ROOT >> /media/ROOT/etc/fstab
    cat fstab
    reboot

    
# GRUB EFI Repair

    setxkbmap fr
    sudo apt update
    sudo apt install grub-efi
    lsblk -fe7
    sudo mount /dev/sda2 /mnt
    sudo mkdir -p /mnt/boot/efi
    sudo mount /dev/sda1 /mnt/boot/efi
    sudo mount --bind /dev /mnt/dev
    sudo mount --bind /proc /mnt/proc
    sudo mount --bind /sys /mnt/sys
    sudo mount --bind /run /mnt/run
    modprobe efivars
    sudo chroot /mnt
    grub-install /dev/sda
    grub-mkconfig -o /boot/grub/grub.cfg
    update-grub
    exit
    sudo umount -R /mnt
    reboot


____________________________________________________________________

###### apt-get install grub-efi-amd64



#### apt-get install grub-efi
#### apt-get remove grub-pc


# GRUB thèmes

>[!NOTE]
>S'assurer que les image soit au format PNG argb

    sudo cp /usr/share/grub/default/grub /etc/default/grub
    echo "GRUB_GFXMODE=1920x1080x32,1024x768x32,640x480,auto" | sudo tee -a /etc/default/grub
    echo "GRUB_THEME=\"/boot/grub/theme/ubuntu/theme.txt"\" | sudo tee -a /etc/default/grub
    sudo sed -i 's/GRUB_TIMEOUT_STYLE=hidden/GRUB_TIMEOUT_STYLE=menu/g' /etc/default/grub
    sudo sed -i 's/GRUB_TIMEOUT=0/GRUB_TIMEOUT=10/g' /etc/default/grub
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    sudo update-grub

    
# Check Firm boot type.
    [ -d /sys/firmware/efi ] && echo "UEFI Boot Detected" || echo "Legacy BIOS Boot Detected"

  
