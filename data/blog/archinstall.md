---
title: Установка Arch Linux
date: '2022-02-16'
tags: ['arch', 'btrfs', 'linux']
draft: false
summary: Описана установка на ssd с btrfs и efi.
layout: PostLayout
---


<TOCInline toc={props.toc} asDisclosure />

---

## Предисловие  

Привет! В Arch Linux нет GUI для установки. Так случилось, потому что у Arch есть принципы, два из которых - [простота](https://wiki.archlinux.org/title/Arch_Linux#Simplicity) и [направленность на пользователя](https://wiki.archlinux.org/title/Arch_Linux#User_centrality).  

Простота в понимании сообщества дистрибутива определена как "отсутствие ненужных дополнений или модификаций". Пользователи, на которых направлен дистрибутив - опытные и/или любознательные.  

Советую воспользоваться [руководством по установке](https://wiki.archlinux.org/title/Installation_guide) c ArchWiki и не следовать инструкции слепо. Хорошо, если вы будите понимать каждый этап установки.   

Позже напишу статью про преимущества btrfs и как ими воспользоваться.


---



## Подготовка диска

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

 - #### Создадим разделы для efi и btrfs

| Раздел      | Размер| Тип             |Код типа в fdisk|
|-------------|-------|-----------------|----------------|
|`/dev/sdX1`  |200 M  |EFI System       |1               |
|`/dev/sdX2`  |100 G  |Linux filesystem |20              |

---

- #### Создадим FAT32 на первом разделе 


```
$ mkfs.fat -F32 /dev/sdX1
```

---

- #### Создадим btrfs на втором разделе

```
mkfs.btrfs -L MAIN /dev/sdX2
```  

Ключ `-L` установит метку диска. Мы ставим её, чтобы потом иметь [доступ к btrfs из Windows.](https://github.com/maharmstone/btrfs)

---

- #### Смонтируем раздел btrfs в `/mnt`

```
mount /dev/sdX2 /mnt
```
`/mnt` - каталог для ручного монтирования файловых систем

---
  
- #### Создадим подразделы на смонтированном btrfs разделе

```
btrfs su cr /mnt/@
```
```
btrfs su cr /mnt/@home
```
`btrfs su cr` - псевдоним для `btrfs subvolume create`. Название подразделов начинаются с `@` чтобы не путать их с другими каталогами.

---

- #### Размонтируем btrfs раздел


```
umount -R /mnt
```

---

- #### Смонтируем подразделы btrfs
```
mount -o subvol=/@,noatime,ssd_spread,compress=zstd:1,discard=async,ssd,commit=600  /dev/sdX2 /mnt
```
```
mkdir /mnt/{boot,home}
```
```
mount -o subvol=/@home,noatime,ssd_spread,compress=zstd:1,discard=async,ssd,commit=600 /dev/sdX2 /mnt/home
```
- #### Смонтируем efi раздел
```
mkdir /mnt/boot/efi
```
```
mount /dev/sdX1 /mnt/boot/efi
```  
 `/mnt` -  системный каталог и в него мы монтируем корневой раздел нашей новой файловой системы. Но другие каталоги нужно создать в `/mnt` перед тем, как можно будет их смонтировать.  

Описание используемых параметров монтирования подразделов btrfs:  

| Параметр   | Описание |
|:-----------|:--|
|`subvol`|имя подраздела btrfs|
|`noatime`|отключает запись времени доступа к файлу|
|`ssd_spread`|запись в пустые области диска, ускорит работу ssd|
|`compress`|режим сжатия [zstd:1](https://btrfs.wiki.kernel.org/index.php/Compression#What_are_the_differences_between_compression_methods.3F)|
|`discard`|будет освобождать неиспользуемые блоки с ssd, `async` группирует блоки для снижения нагрузки|
|`ssd`|оптимизации для ssd|
|`commit`|задаст интервал записи данных в файловую систему в секундах|

---

## Установка системы

---
- #### Установим систему в корневой каталог

```
pacstrap /mnt base base-devel linux-zen linux-zen-headers linux-firmware networkmanager vim git intel-ucode iucode-tool
```  

`pacstrap` скрипт для установки пакетов в новый корневой каталог

Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|:--|
|`base`|[группа пакетов](https://archlinux.org/packages/core/any/base/) для базовой установки Arch Linux|
|`base-devel`|[группа пакетов](https://archlinux.org/groups/x86_64/base-devel/) с инструментами для сборки|
|`linux-zen`|[ядро](https://wiki.archlinux.org/title/Kernel_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)) linux-zen|
|`linux-zen-headers`|заголовки ядра linux-zen|
|`linux-firmware`|драйвера устройств|
|`networkmanager`|[набор инструментов](https://wiki.archlinux.org/title/NetworkManager) для настройки сети|
|`vim`|камсольный текстовые редактор|
|`git`|[интерфейс для AUR](https://wiki.archlinux.org/title/git)|
|`intel-ucode`|[обновление микрокода процессора](https://wiki.archlinux.org/title/Microcode), если у в AMD используйте `amd-ucode`|
|`iucode-tool`||[управление обновлениями микрокода](https://gitlab.com/iucode-tool/iucode-tool/-/wikis/home)|

---

- #### Запишем информацию созданных нами файловых системах в `/etc/fstab`

```
genfstab -U /mnt >> /mnt/etc/fstab
```
Ключ `-U` создаст fstab основанный на UUID  

---

## Базовая настройка
---

- #### Сменим корневой каталог на каталог с новой системой
```
arch-chroot /mnt
```
Сейчас система установлена на диск. Если установка привела к ошибке, то можно будет зайти в установленную систему через `arch-root` после перезагрузки. Не забудьте смонтировать файловые системы. При монтировании btrfs можно будет указать только параметр `subvol`.

---
- #### Добавим имя новой системы и свяжем его с localhost
```
echo metropolis > /etc/hostname
```
```
echo -e "127.0.0.1 localhost\n::0 localhost\n127.0.0.1 metropolis" >> /etc/hosts
```
---
- #### Включим службу NetworkManager
```
systemctl enable NetworkManager
```
```
systemctl mask NetworkManager-wait-online.service
```
`systemctl` - основная команда для управления [systemd](https://wiki.archlinux.org/title/systemd).  
`mask` - делает невозможным запуск службы.  
`NetworkManager-wait-online.service` - служба сетевого запуска. Ее отключение ускорит загрузку.  

---

- #### Настроим время

```
timedatectl set-timezone Europe/Moscow
```
```
timedatectl set-ntp true
```

---
- #### Установим локаль
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
- #### Установим пароль root
```
passwd
```
---
- #### Создадим пользователя
```
useradd -m sonic
```
ключ `-m` создаст пользовательский каталог `/home/sonic`
```
passwd sonic
```
```
echo "sonic ALL=(ALL:ALL) ALL" >> /etc/sudoers
```
```
visudo -c
```  
`/etc/sudoers` - файл настроек [`sudo`](https://wiki.archlinux.org/title/Sudo). Руководство по `sudo` говорит, что его следует [всегда](https://wiki.archlinux.org/title/Sudo#Using_visudo) редактировать с помощью команды `visudo`. Давайте немного побудем бунтарями и просто выполним проверку `/etc/sudoers` с помощью `visudo -с`.


---
## Настройка загрузчика
---
- #### Установим пакеты загрузчика
```
pacman -S grub efibootmgr
```  

Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|:--|
|`grub`|[загрузчик ядра](https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader)|
|`efibootmgr`|[редактор загрузочных записей efi](https://man.archlinux.org/man/efibootmgr.8.en)|

---
- #### Установим grub в `boot/efi`
```
grub-install --efi-directory=/boot/efi
```
---
- #### Запишем файл конфигурации grub
```
grub-mkconfig -o /boot/grub/grub.cfg
```  
---
- #### Размонтируем разделы и перезагрузимся в новую систему
```
exit
```
```
umount -R /mnt
```
```
reboot
```  

Теперь система установлена. Дальше я рекомендую прочитать [общии рекомендации](https://wiki.archlinux.org/title/General_recommendations).  

Позже добавлю статьи по настройке Arch Linux после установки и по работе с btrfs.  

Надеюсь, эта статья смогла вам помочь! Если у вас остались вопросы, то я могу ответить на них в комментариях.