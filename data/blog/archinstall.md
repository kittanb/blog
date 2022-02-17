---
title: Установка Archlinux
date: '2022-02-16'
tags: ['arch', 'btrfs', 'linux']
draft: false
summary: на btrfs ssd efi
---

`Статья находится в разработке`

### Проверим режим загрузки

```
ls /sys/firmware/efi/efivars
```
### Создадим на диске таблицу GPT и разобьем его на разделы с btrfs и efi


| Раздел    | Размер  | Тип  |
|-----------|---------|------|
|`/dev/sdX1`| 200 МиБ | ef00 |
|`/dev/sdX2`| 20+ ГиБ | 8300 |

### Создадим файловую систему

```
mkfs.fat -F32 /dev/sdX1
```
```
mkfs.btrfs -L MAIN dev/sdX2
```

### Примонтируем btrfs и перейдем в нее

```
mount /dev/sdX2 /mnt
```
```
cd /mnt
```

### Создадим subvolumes на примонтированном btrfs разделе

```
btrfs su cr @
```
```
btrfs su cr @home
```

### Выйдем и отмонтируем btrfs раздел

```
cd
```
```
umount /mnt
```
### Смонтируем subvolumes btrfs и efi раздел

```
mount -o subvol=/@,defaults,noatime,autodefrag,compress=zstd,discard=async,ssd,commit=120  /dev/sdX2 /mnt
```
```
mkdir /mnt/{boot,home}
```
```
mkdir /mnt/boot/efi
```
```
mount -o subvol=/@home,defaults,noatime,autodefrag,compress=zstd,discard=async,ssd,commit=120 /dev/sdX2 /mnt/home
```
```
mount /dev/sdX1 /mnt/boot/efi
```
### Устанавим систему в корневой каталог
```
pacstrap /mnt base base-devel linux-zen linux-zen-headers linux-firmware dhcpcd zsh vim git intel-ucode
```
### Создадим файл fstab в корневом каталоге
```
genfstab -U /mnt >> /mnt/etc/fstab
```
### Сменим корневой каталог на тот, что установили
```
arch-chroot /mnt
```
### Добавим hostname и свяжем с localhost
```
echo metropolis > /etc/hostname
```
```
echo -e "127.0.0.1 localhost\n::0 localhost\n127.0.0.1 metropolis" >> /etc/hosts
```
### Включим службу dhcpcd
```
systemctl enable dhcpcd
```
### Локализируем
```
sed -i "s/#en_US.UTF-8/en_US.UTF-8/g" /etc/locale.gen
```
```
locale-gen
```
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```
### Установим пароль root и создадим пользователя
```
useradd -m -G wheel -s /bin/zsh sonic
```
```
passwd sonic
```
```
passwd
```
```
echo "sonic ALL=(ALL:ALL) ALL" >> /etc/sudoers
```
```
visudo -c
```
### Установим загрузчик
```
pacman -S grub efibootmgr os-prober
```
```
grub-install --efi-directory=/boot/efi
```
```
sed -i "s/#GRUB_DISABLE_OS_PROBER=false/GRUB_DISABLE_OS_PROBER=false/g" /etc/default/grub
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```
### Размонтируем разделы и перезагрузим систему
```
exit
```
```
umount -R /mnt
```
```
shutdown -r now
```
