---
title: Установка Arch Linux
date: '2022-02-16'
tags: ['ar ch', 'btrfs', 'linux']
draft: false
summary: Описана установка на ssd с btrfs и efi.
layout: PostLayout
---

`Статья находится в разработке`

<TOCInline toc={props.toc} asDisclosure />

## Предисловие  

Привет! В Arch Linux нет GUI для установки. Так случилось, потому что у Arch есть принципы, два из которых - [простота](https://wiki.archlinux.org/title/Arch_Linux#Simplicity) и [направленность на пользователя](https://wiki.archlinux.org/title/Arch_Linux#User_centrality).  

Простота в понимании сообщества определена как "отсутствие ненужных дополнений или модификаций". Пользователи, на которых направлен дисрибутив - опытные или любознательные.  

Я советую руководствоваться [гайдом по установке](https://wiki.archlinux.org/title/Installation_guide) c ArchWiki и не следовать инструкции слепо. Хорошо, если вы будите понимать каждый этап установки. Тут описан случай установки Arch Linux на ssd с efi и btrfs.

---

## Подготовка

---

### Разметка диска

Для редактирования разделов используем [fdisk](https://wiki.archlinux.org/title/Fdisk).  

`fdisk -l` покажет список устройств.  `fdisk /dev/sdX` запустит диалог для ввода команд.  

Я назвал диск `/dev/sdX` чтобы избежать путаницы. Название вашего диска будет отличаться.

Ключи диалогового окна fdisk:

| Ключ       |Функция   |
|:-----------|:---------|
|`m`| покажет справку   |
|`g`| создаст новую GPT |
|`d`| удалит раздел     |
|`n`| создаст раздел    |
|`t`| сменит тип раздела|
|`w`| запишет изменения |
|`q`| закроет диалог    |

- Создадим разделы для efi и btrfs:

| Раздел      | Размер| Тип             |Код типа в gdisk|
|-------------|-------|-----------------|----------------|
|`/dev/sdX1`  |200 M  |EFI System       |1               |
|`/dev/sdX2`  |100 G  |Linux filesystem |20              |

---

### Создание файловой системы  

---

- Создадим фаловую систему FAT32 для раздела с efi:  

```
mkfs.fat -F32 /dev/sdX1
```

- Создадим btrfs. Устновим метку диска `MAIN`:

```
mkfs.btrfs -L MAIN dev/sdX2
```

---

- Примонтируем btrfs:

```
mount /dev/sdX2 /mnt
```
```
cd /mnt
```
`/mnt` - каталог для ручного монтирования фаловых систем

---
  
- Создадим подразделы на примонтированном btrfs разделе:

```
btrfs su cr @
```
```
btrfs su cr @home
```
`btrfs su cr` - псевдоним для `btrfs subvolume create`. Название подразлов начинаются с `@` чтобы не путать их с другими каталогами.

---

- Выйдем из /mnt и отмонтируем btrfs раздел:

```
cd
```
```
umount /mnt
```

---

- Смонтируем подразделы btrfs и efi раздел:
```
mount -o subvol=/@,noatime,autodefrag,compress=zstd:1,discard=async,ssd,commit=120  /dev/sdX2 /mnt
```
```
mkdir /mnt/{boot,home}
```
```
mkdir /mnt/boot/efi
```
```
mount -o subvol=/@home,noatime,autodefrag,compress=zstd:1,discard=async,ssd,commit=120 /dev/sdX2 /mnt/home
```
```
mount /dev/sdX1 /mnt/boot/efi
```  
 `/mnt` -  системный каталог и в него мы монтируем корневой раздел нашей новой файловой системы. Но другие каталоги нужно создать в `/mnt` перед тем, как можно будет их монтировать.  

Описание используемых параметров монитрования подразделов btrfs:  

| Параметр   | Описание |
|:-----------|:--|
|`subvol`|имя подраздела btrfs|
|`noatime`|отключает запись времени доступа к файлу|
|`autodefrag`|включит автоматическу дефрагментацию|
|`compress`|режим сжатия `zstd:1`|
|`discard`|будет освобождать неиспользуемые блоки с ssd, `async` группирует блоки для снижения нагрузки|
|`ssd`|оптимизации для ssd|
|`commit`|задаст интервал записи данных в файловую систему в секундах|

---

## Установка системы

---
- Устанавим систему в корневой каталог:

`pacstrap` скрипт для установки пакетов в новый корневой каталог

```
pacstrap /mnt base base-devel linux-zen linux-zen-headers linux-firmware dhcpcd vim
```

Список устанавливаемых мною пакетов
| Пакет   | Описание |
|:-----------|:--|
|`base`|[группа пакетов для базовой установки Arch Linux](https://archlinux.org/packages/core/any/base/)|
|`base-devel`|[группа пакетов с инструментами для сборки](https://archlinux.org/groups/x86_64/base-devel/)|
|`linux-zen`|ядро linux-zen|
|`linux-zen-headers`|заголовки ядра linux-zen|
|`linux-firmware`|драйвера устройств|
|`dhcpcd`|dhcp клиент|
|`vim`|текстовые редактор|

- Запишем информацию созданных нами файловых системах в /etc/fstab:

```
genfstab -U /mnt >> /mnt/etc/fstab
```
Ключ `-U` создаст fstab основанные на UUID  

---

## Начальная настрока
---

- Сменим корневой каталог на тот, что установили:
```
arch-chroot /mnt
```
Сейчас данные записаны на диск. Если установка привела к ошибке, то можно будет зайти в arch-root после перезагрузки, но не забудьте примонтировать файловые системы. При монтировании btrfs можно будет указать только параметр `subvol`.

---
- Добавим hostname и свяжем с localhost:
```
echo metropolis > /etc/hostname
```
```
echo -e "127.0.0.1 localhost\n::0 localhost\n127.0.0.1 metropolis" >> /etc/hosts
```
---
- Включим службу dhcpcd:
```
systemctl enable dhcpcd
```
---
- Создадим локализацию:
```
sed -i "s/#en_US.UTF-8/en_US.UTF-8/g" /etc/locale.gen
```
```
locale-gen
```
```
echo LANG=en_US.UTF-8 > /etc/locale.conf
```
---
- Установим пароль root:
```
passwd
```
---
- Создадим пользователя:
```
useradd -m -G wheel -s /bin/zsh sonic
```
```
passwd sonic
```
```
echo "sonic ALL=(ALL:ALL) ALL" >> /etc/sudoers
```
```
visudo -c
```
---
- Установим загрузчик:
```
pacman -S grub efibootmgr intel-ucode os-prober
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
---
- Размонтируем разделы и перезагрузим систему:
```
exit
```
```
umount -R /mnt
```
```
shutdown -r now
```
