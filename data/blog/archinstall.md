---
title: Установка Arch Linux
date: '2022-02-16'
tags: ['arch', 'btrfs']
draft: false
summary: Установим Arch Linux на реальный комп с btrfs и efi.
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
pacstrap /mnt base base-devel linux-zen linux-zen-headers linux-firmware networkmanager intel-ucode iucode-tool vim git curl wget
```  

`pacstrap` скрипт для установки пакетов в новый корневой каталог

Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|:--|
|[base](https://archlinux.org/packages/core/any/base/)|группа пакетов для базовой установки Arch Linux|
|[base-devel](https://archlinux.org/groups/x86_64/base-devel/)|группа пакетов с инструментами для сборки|
|[linux-zen](https://wiki.archlinux.org/title/Kernel)|ядро linux-zen|
|[linux-zen-headers](https://archlinux.org/packages/extra/x86_64/linux-zen-headers/)|заголовки ядра linux-zen|
|[linux-firmware](https://archlinux.org/packages/core/any/linux-firmware/)|драйвера устройств|
|[networkmanager](https://wiki.archlinux.org/title/NetworkManager)|набор инструментов для настройки сети|
|[intel-ucode](https://wiki.archlinux.org/title/Microcode)|обновление микрокода процессора, если у в AMD используйте `amd-ucode`|
|[iucode-tool](https://gitlab.com/iucode-tool/iucode-tool/-/wikis/home)|управление обновлениями микрокода|
|[vim](https://wiki.archlinux.org/title/vim)|камсольный текстовые редактор|
|[git](https://wiki.archlinux.org/title/git)|интерфейс для AUR|
|[curl](https://wiki.archlinux.org/title/CURL)|инструмент для передачи данных с помощью URL-адресов|
|[wget](https://wiki.archlinux.org/title/Wget)|инструмент для загрузки файлов по http и ftp|

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

---  

После перезагрузки мы попадем в нашу свежую систему.    

На ArchWiki есть статья с [общими рекомендациями](https://wiki.archlinux.org/title/General_recommendations) после установки системы. Если прочитать её, то можно понять, что еще нам предстоит настроить.  

В [следующей статье](https://www.kittan.ru/blog/gui) мы установим графический интерфейс и подготовлю систему к работе.  

Надеюсь, эта статья была полезна вам! А если у вас возникла проблема, вы можете рассказать о ней в комментариях. Я обязательно отвечу.