---
title: Рабочий стол на Arch
date: '2022-03-17'
tags: ['arch', 'nvidia', 'aur', 'btrfs']
draft: false
summary: Подготовим Arch в качестве десктопа для работы и игорь.
images: []
---

<TOCInline toc={props.toc} asDisclosure />

---

Привет! В этой статье мы подготовим наш [свежий Arch](https://www.kittan.ru/blog/archinstall) к работе в качестве домашнего десктопа.  

Мы используем:  

- рабочий стол `KDE Plasma` или `GNOME`
- проприетарный DKMS драйвер `NVIDIA`  
- `yay` (майонез)  
- `zramd` в качестве SWAP
- `nftables` как брандмауэр
- `timeshift` для резервного копирования  

---

## Подготовка  

- #### Включим 32-битный репозиторий  

Нам нужны будут некоторые зависимости оттуда.  
Для этого в `/etc/pacman.conf` раскомментируем репозиторий multilib:  

```
[multilib]
Include=/etc/pacman.d/mirrorlist
```  

Если хотите получить доступ к тестовой версии Plasma, то в начале списка репозиториев (перед записью [testing]) в `/etc/pacman.conf` добавьте:  

```
[kde-unstable]
Include = /etc/pacman.d/mirrorlist
```  

А если к тестовой версии Gnome, то:  

```
[gnome-unstable]
Include = /etc/pacman.d/mirrorlist
```  

Раз мы зашли сюда, раскомментируем еще и `Color` c `ParallelDownloads = 5` в разделе `Misc options`

```
# Misc options
#UseSyslog
Color
#NoProgressBar
CheckSpace
#VerbosePkgLists
ParallelDownloads = 5
```
Это включит параллельную загрузку и подсветку в терминале.

---  

- #### Обновим систему  

```
sudo pacman -Suy
```

---  

## Установка среды рабочего стола  

Сейчас я использую GNOME и KDE Plasma, поэтому опишу их установку.  

На мой взгляд, основные отличия Gnome от Plasma:  

- Gnome полностью построен на GTK, а Plasma на Qt
- В дизайне интерфейса для Gnome на первом месте пользовательский опыт, а для Plasma - функциональность
- Gnome использует новый графический сервер Wayland по умолчанию и неплохо с ним работает. У KDE с Wayland всё еще могут [быть](https://community.kde.org/KWin/Wayland) какие-то проблемы. А могут и [не быть](https://www.phoronix.com/scan.php?page=news_item&px=Qt-Wayland-NVIDIA-Thread), лучше проверьте работу своей системы на Plasma сперва с Wayland.  

> P.S У меня 1050 и с проприетарным драйвером wayland работает пока плохо. Хотя, возможно, у меня что-то плохое с руками

Выберете полную или минимальную установку этих [DE](https://wiki.archlinux.org/title/desktop_environment).  

Если вы еще не очень разбираетесь - ставьте полные группы пакетов и удаляйте ненужное. Такой подход уменьшит количество вероятных ошибок в работе DE. В противном случае ставьте минимальную версию и добавляйте нужные вам пакеты из полных групп.  

---  

- #### Установим Gnome  

GNOME использует [Wayland](https://wiki.archlinux.org/title/wayland) по умолчанию.  

- Минимальная установка:
```
sudo pacman -S gnome-shell gnome-terminal gnome-tweaks gnome-control-center xdg-user-dirs gdm gnome-keyring nautilus eog file-roller
```  

- Полная установка:
```
sudo pacman -S gnome gnome-extra
```

---  

- #### и включим GDM  

```
sudo systemctl enable gdm
```

---

Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|:--|
|[gnome-shell](https://archlinux.org/packages/extra/x86_64/gnome-shell/)|оболочка рабочего стола Gnome|
|[gnome-terminal](https://archlinux.org/packages/extra/x86_64/gnome-terminal/)|эмулятор терминала|
|[gnome-tweaks](https://archlinux.org/packages/extra/any/gnome-tweaks/)|интерфейс для расширенных настроек Gnome|
|[gnome-control-center](https://archlinux.org/packages/extra/x86_64/gnome-control-center/)|интерфейс для основных настроек Gnome|
|[xdg-user-dirs](https://wiki.archlinux.org/title/XDG_user_directories)|менеджер пользовательских каталогов|
|[gdm](https://wiki.archlinux.org/title/GDM)|менеджер дисплея Gnome|
|[gnome-keyring](https://wiki.archlinux.org/title/GNOME/Keyring)|хранитель паролей|
|[nautilus](https://wiki.archlinux.org/title/GNOME/Files)|файловый менеджер|
|[eog](https://archlinux.org/packages/extra/x86_64/eog/)|просмотр изображений|
|[file-roller](https://archlinux.org/packages/extra/x86_64/file-roller/)|архиватор|
|[gnome](https://archlinux.org/groups/x86_64/gnome/)|группа пакетов с десктопом и основными приложениями|
|[gnome-extra](https://archlinux.org/groups/x86_64/gnome-extra/)|группа пакетов с дополнительными приложениями|  

---  

- #### Или установим KDE Plasma  

KDE установим с Xorg.  

- Минимальная установка:
```
sudo pacman -S xorg-server xorg-apps plasma-desktop sddm plasma-nm plasma-pa dolphin konsole kdeplasma-addons kde-gtk-config
```  

- Полная установка:
```
sudo pacman -S xorg-server xorg-apps plasma kde-applications
```  

---  

Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|--|
|[xorg-server](https://wiki.archlinux.org/title/Xorg#Installation)|Xorg сервер|
|[xorg-apps](https://archlinux.org/groups/x86_64/xorg-apps/)|группа пакетов с конфигами для Xorg|
|[plasma-desktop](https://wiki.archlinux.org/title/KDE#Installation)|оболочка рабочего стола Plasma|
|[sddm](https://wiki.archlinux.org/title/SDDM)|менеджер дисплея KDE|
|[plasma-nm](https://archlinux.org/packages/extra/x86_64/plasma-nm/)|апплет Plasma для NetworkManager|
|[plasma-pa](https://archlinux.org/packages/extra/x86_64/plasma-pa/)|апплет Plasma для PulseAudio|
|[dolphin](https://wiki.archlinux.org/title/Dolphin)|файловый менеджер|
|[konsole](https://wiki.archlinux.org/title/Konsole)|эмулятор терминала|
|[kdeplasma-addons](https://archlinux.org/packages/extra/x86_64/kdeplasma-addons/)|улучшения для Plasma|
|[kde-gtk-config](https://archlinux.org/packages/?name=kde-gtk-config)|интеграция с GTK приложениями|
|[plasma](https://archlinux.org/groups/x86_64/plasma/)|группа пакетов с десктопом и основными приложениями|
|[kde-applications](https://archlinux.org/groups/x86_64/kde-applications/)|группа пакетов с группами дополнительных приложениями|

Вместо kde-application можно выбрать только нужные вам группы [тут](https://archlinux.org/packages/extra/any/kde-applications-meta/) или [тут](https://archlinux.org/packages/kde-unstable/any/kde-applications-meta/)  

---  

- #### и включим SDDM  

```
sudo systemctl enable sddm
```

---  

## Установка драйвера NVIDIA  

Установим проприетарный DKMS драйвер NVIDIA. Иногда запускаю игры и мне нужна производительность. А DKMS позволит нам не возиться с модулями ядра при обновлении.  

- #### Установим драйвер NVIDIA и Vulkan  

```
sudo pacman -S nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader opencl-nvidia lib32-opencl-nvidia libxnvctrl
```
Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|--|
|[nvidia-dkms](https://archlinux.org/packages/extra/x86_64/nvidia-dkms/)|[DKMS](https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support) проприетарный драйвер NVIDIA|
|[nvidia-utils](https://archlinux.org/packages/extra/x86_64/nvidia-utils/)|утилиты драйвера NVIDIA|
|[lib32-nvidia-utils](https://archlinux.org/packages/multilib/x86_64/lib32-nvidia-utils/)|утилиты драйвера NVIDIA (32-bit)|
|[nvidia-settings](https://wiki.archlinux.org/title/NVIDIA#nvidia-settings)|редактор опций NVIDIA|
|[vulkan-icd-loader](https://wiki.archlinux.org/title/Vulkan)|графический API Vulkan|
|[lib32-vulkan-icd-loader](https://archlinux.org/packages/multilib/x86_64/lib32-vulkan-icd-loader/)|графический API Vulkan (32-bit)|
|[opencl-nvidia](https://wiki.archlinux.org/title/GPGPU#OpenCL)|среда выполнения OpenCL для NVIDIA|
|[lib32-opencl-nvidia](https://archlinux.org/packages/multilib/x86_64/lib32-opencl-nvidia/)|среда выполнения OpenCL для NVIDIA (32-bit)|
|[libxnvctrl](https://archlinux.org/packages/extra/x86_64/libxnvctrl/)|API для NVIDIA и X|

---  

- #### Добавим модули ядра для NVIDIA и brtfs

Отредактируем скрипт Initial ramdisk `/etc/mkinitcpio.conf`.  
В строку `MODULES` добавим `nvidia nvidia_modeset nvidia_uvm nvidia_drm crc32c libcrc32c zlib_deflate btrfs`.  

Теперь пересобираем образ ядра:  
```
sudo mkinitcpio -P
```
Обновляем загрузчик:  
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
```
reboot
```  

---

- #### Настроим драйвер NVIDIA  

Сперва сгенерируем файл конфигурации X сервера:  

```
sudo nvidia-xconfig
```
```
reboot
```

Теперь запустим настройки NVIDIA  

```
sudo nvidia-settings
```

Во вкладке `X Server XVideo Settings` выберем основной монитор.  
Во вкладке `PowerMizer` в разделе `PowerMizer Settings` выберем `Prefer Maximum Performance`.  
Во вкладке `X Server Display Configuration` выберем наше разрешение и частоту и сохраним `Save to X Configuration File`.  
Запустим `nvidia-settings` без `sudo` и повторим всё настройки выше. Но не будем сохранять через `Save to X Configuration File`.  

---  


## Начальная настройка  

- #### Установим yay  

Одно из главных преимуществ Arch Linux - это Arch User Repository (AUR). В пользовательских репозиториях очень быстро появляются новые версии пакетов.  

Скрипты с информацией о сборке пакетов тут неофициальные. У меня никогда не возникало проблем и я ничего не слышал о взломах через AUR, но будьте благоразумны.  

[yay](https://github.com/Jguer/yay) - один из помощников AUR. С его помощью можно устанавливать и обновлять пакеты из AUR и обычных репозиториев.  

Использование без ключей выполнит поиск пакета, содержащего искомые слова в названии или описании. Поиск идет по подключенным репозиториям и в AUR.  

Создадим каталог для git и перейдём в него. Я сделаю это в Download:  

```
mkdir ~/Download/git
```
```
cd Download/git/
```
Клонируем репозиторий с yay и установим его с помощью [makepkg](https://wiki.archlinux.org/title/Makepkg):  

```
git clone https://aur.archlinux.org/yay.git
```
```
cd yay
```
```
makepkg -sric
```
`-i` - установит пакет после сборки  
`-s` - установит недостающие зависимости  
`-r` - удалит зависимости для сборки после ее окончания  
`-c` - очистит каталог установки  

---  

- #### Установим брандмауэр  

Установим и запустим службу `nftables':    

```
sudo pacman -S nftables
```
```
sudo systemctl enable --now nftables
```

Нас интересует простой брандмауэр. В `nftables` есть дефолтная конфигурация, которая лежит в `/etc/nftables.conf`. Правила из этого файла загружаются при запуске службы `nftables.service`, значит ничего больше нам делать не нужно.  

---  

- #### Настроим файл подкачки  

`zramd` - служба, создающая файл подкачки в памяти. Я использую её вместо файла подкачки.

Установим и запустим службу `zramd`:

```
yay zramd
```
```
sudo systemctl enable --now zramd
```

Теперь вывод `lsblk` покажет нам, что в SWAP смонтирован и его объем равен оперативной памяти.  

---  

- #### Настроим резервное копирование  

Если вы собираетесь ставить всякие странные вещи в свою систему, то лучше позаботиться о резервном копировании.   

В [предыдущей статье](https://www.kittan.ru/blog/archinstall) мы установили систему на диск с `btrfs`. Одна из причин, по которой мы выбрали `btrfs` - простое создание и восстановление снапшотов. Они отличаются от бэкапов тем, что сохраняют только 1 полную резервную копию. А дальше записывают лишь изменения, произошедшии с данными резервируемых подразделов. Так мы экономим время и ресурсы системы.  

Самое простое решение для резервного копирования `btrfs` с GUI - это `timeshift`. Он позволит нам восстановиться из GRUB, с рабочего стола или загрузившись с live системы через графический интерфейс.  

`timeshift` создает снапшоты подразделов `btrfs` на новом подразделе.  

Сперва установим `timeshift`:  
```
yay timeshift
```
После установки запустим `timeshift` из меню приложений нашей графической оболочки. Откроется простой мастер настройки резервного копирования.  

- В разделе `Select Snapshot Type` выберем `BTRFS`
- В разделе `Select Snapshot Location` выберем наш раздел с`BTRFS`
- В разделе `Select Snapshot Levels` выберем нужное нам расписание
- В разделе `User Home Directories` оставим пустым чекбокс `Include @home su in backup`
- Теперь жмем `Finish`
- В открывшимся интерфейсе `TimeShift` нажмем `Create`. Это запустит создание первого снапшота.  

Теперь, чтобы не случилось, мы можем вернуться в это место.  

Обратите внимание, мы НЕ сохраняем наш EFI раздел, но восстановить или создать заново его можно из live системы. 

---

Теперь установим [timeshift-autosnap](https://gitlab.com/gobonja/timeshift-autosnap). Это скрипт автоматического создания снапшота перед обновлением системы. 
```
yay timeshift-autosnap
```
Изменить настройки создания снапшотов при обновлении можно в файле `/etc/timeshift-autosnap.conf`. Скрипт добавит в GRUB раздел с вариантами загрузки системы из созданного им снапшота.

---  

Ура! Наша система почти готова. Но уже сейчас ей можно пользоваться не боясь прострелить себе колено.

В [следующей статье](https://www.kittan.ru/blog/improve) мы оптимизируем и украсим нашу систему. 

Надеюсь, эта статья была полезна вам! А если у вас возникла проблема, вы можете рассказать о ней в комментариях. Я обязательно отвечу.

