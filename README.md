

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
    cat /proc/cmdline
    cat /proc/partitions 
    blkid
    lsblk
    set prefix
    mount -o remount,ro /root
    fsck -f /dev/sda3 -y 
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

    setxkbmap fr
    sudo apt update
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

###### apt-get install grub-efi-amd64 << EFI_amd64
###### apt-get install grub-efi << EFI
###### apt-get remove grub-pc << LEGACY


# GRUB thèmes

>[!NOTE]
>S'assurer que les image soit au format PNG argb

    sudo cp /usr/share/grub/default/grub /etc/default/grub
    echo "GRUB_GFXMODE=1920x1080x32,1024x768x32,640x480,auto" | sudo tee -a /etc/default/grub
    echo "GRUB_THEME=\"/boot/grub/themes/trixie/theme.txt"\" | sudo tee -a /etc/default/grub
    sudo sed -i 's/GRUB_TIMEOUT_STYLE=hidden/GRUB_TIMEOUT_STYLE=menu/g' /etc/default/grub
    sudo sed -i 's/GRUB_TIMEOUT=0/GRUB_TIMEOUT=10/g' /etc/default/grub
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    sudo update-grub


# Reinstall grub après une purge.


```bash
sudo rm -f /var/lib/dpkg/lock-frontend /var/lib/dpkg/lock /var/cache/apt/archives/lock
sudo mkdir -p /etc/grub.d
sudo mv /etc/default/grub /etc/default/grub.bak
cat <<EOF | sudo tee /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=\`lsb_release -i -s 2> /dev/null || echo Debian\`
GRUB_CMDLINE_LINUX_DEFAULT="splash"
GRUB_CMDLINE_LINUX=""
GRUB_ENABLE_UEFI_FIRMWARE_SETUP="true"
GRUB_DISABLE_OS_PROBER="false"
EOF

cat <<EOF | sudo tee /etc/grub.d/40_custom
#!/bin/sh
exec tail -n +3 \$0
menuentry 'UEFI Firmware Settings' \$menuentry_id_option 'uefi-firmware' {
	fwsetup
}
EOF

sudo apt update -y
sudo apt purge --allow-remove-essential memtest86+ -y
sudo rm -f /var/lib/dpkg/info/memtest86+.*
sudo dpkg --remove --force-all memtest86+
sudo apt purge --allow-remove-essential grub-common grub2-common grub-efi-amd64-bin grub-efi-amd64-signed shim-signed -y
sudo apt install --reinstall -y grub-common grub2-common grub-efi-amd64-bin grub-efi-amd64-signed shim-signed efibootmgr
sudo find /etc/grub.d/ -type f -not -name "00_header" -not -name "10_linux" -not -name "20_linux_xen" -not -name "30_os-prober" -not -name "40_custom" -not -name "41_custom" -delete
sudo chmod 755 /etc/grub.d/*
sudo apt -f install -y
sudo dpkg --configure -a
sudo grub-install --target=x86_64-efi --recheck
sudo update-grub
```
    
# Check Firm boot type.
    [ -d /sys/firmware/efi ] && echo "UEFI Boot Detected" || echo "Legacy BIOS Boot Detected"

  
