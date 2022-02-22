---
title: Установка Arch Linux
date: '2022-02-16'
tags: ['arch', 'btrfs', 'linux']
draft: false
summary: Описана установка на ssd с btrfs и efi.
layout: PostLayout
---

`Статья находится в разработке`

<TOCInline toc={props.toc} exclude="Overview"/>

## Предисловие

Рекомендую руководствоваться [гайдом по установке](https://wiki.archlinux.org/title/Installation_guide) c ArchWiki. 
Тут описан случай установки Arch Linux на мой комп с ssd и uefi.

## Подготовка файловой системы

### Разметим диск

Для редактирования разделов используем [fdisk](https://wiki.archlinux.org/title/Fdisk). 
'fdisk -l' покажет список устройств. 
'fdisk /dev/sdX' запустит диалог для ввода команд. Я назвал диск '/dev/sdX' чтобы избежать путаницы. Название вашего диска будет отличаться.

Ключи диалогового окна fdisk:

| Ключ      | Функция |
|-----------|---------|
|`m`| покажет справку |
|`g`| создаст новую GPT и уничтожит данные на диске |
|`d`| удалит раздел |
|`n`| создаст раздел |
|`t`| сменит тип раздела |
|`w`| запишет изменения |

Создадим разделы для efi и btrfs:

| Раздел      | Размер | Тип      |Код типа в gdisk|
|-------------|--------|----------|----------------|
|'/dev/sdX1'  |200 M   |EFI System|1               |
|'/dev/sdX1'  |100 G   |Linux fs  |20              |

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
mount -o subvol=/@,noatime,autodefrag,compress=zstd,discard=async,ssd,commit=120  /dev/sdX2 /mnt
```
```
mkdir /mnt/{boot,home}
```
```
mkdir /mnt/boot/efi
```
```
mount -o subvol=/@home,noatime,autodefrag,compress=zstd,discard=async,ssd,commit=120 /dev/sdX2 /mnt/home
```
```
mount /dev/sdX1 /mnt/boot/efi
```

## Установка и первичная настройка системы.


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
