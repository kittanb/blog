---
title: Рабочий стол на Arch
date: '2022-03-17'
tags: ['arch', 'gnome', 'KDE', 'linux']
draft: false
summary: Подготовим Arch в качестве десктопа для работы и игорь
images: []
---

`Статья в разработке`

<TOCInline toc={props.toc} asDisclosure />

Привет! Сегодня мы подготовим наш [свежий Arch](https://www.kittan.ru/blog/archinstall) к работе в качестве домашнего десктопа.    
Мы используем:  

- проприетарный DKMS драйвер NVIDIA  
- рабочий стол KDE или GNOME
- zsh и p10k
- yay (майонез)

## Подготовка
---

- #### Включим 32-битный репозиторий  

Нам нужны будут некоторые зависимости оттуда.  
Для этого в `/etc/pacman.conf` раскомментируем репозиторий multilib:  

```
[multilib]
Include=/etc/pacman.d/mirrorlist
```  

Если хотите получить доступ к тестовой версии Plasma, то в конце `/etc/pacman.conf` добавьте:  

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

- #### Установим yay  

Одно из главных преимуществ Arch Linux - это Arch user repository. В пользовательских репозиториях очень быстро появляются новые версии пакетов.  

Скрипты с информацией о сборке пакетов тут не официальные. У меня никогда не возникало проблем и я ничего не слышал о взломах через AUR, но будьте благоразумны.  

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
makepkg -si
```
`-i` - установит пакет после сборки  
`-s` - установит недостающие зависимости

---  

## Установка и настройка драйвера NVIDIA

- #### Установим драйвер NVIDIA и Vulkan
```
sudo pacman -S nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader opencl-nvidia lib32-opencl-nvidia libxnvctrl
```
Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|--|
|`nvidia-dkms`|[DKMS](https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support) проприетарный драйвер NVIDIA|
|`nvidia-utils`|утилиты драйвера NVIDIA|
|`lib32-nvidia-utils`|утилиты драйвера NVIDIA (32-bit)|
|`nvidia-settings`|[редактор опций NVIDIA](https://wiki.archlinux.org/title/NVIDIA#nvidia-settings)|
|`vulkan-icd-loader`|[графический API Vulkan](https://wiki.archlinux.org/title/Vulkan)|
|`lib32-vulkan-icd-loader`|графический API Vulkan (32-bit)|
|`opencl-nvidia`|[среда выполнения OpenCL](https://wiki.archlinux.org/title/GPGPU#OpenCL) для NVIDIA|
|`lib32-opencl-nvidia`|среда выполнения OpenCL для NVIDIA (32-bit)|
|`libxnvctrl`|API для NVIDIA и X|

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

---  



## Установка DE

Сейчас я использую GNOME и KDE Plasma, поэтому опишу их установку.  

На мой взгляд, основные отличия Gnome от Plasma:  

- Gnome полностью построен на GTK, а Plasma на Qt
- В дизайне интерфейса для Gnome на первом месте пользовательский опыт, а для Plasma - функциональность
- Gnome использует новые графический сервер Wayland по умолчанию и неплохо с ним работает. У KDE с Wayland всё еще могут [быть](https://community.kde.org/KWin/Wayland) какие-то проблемы. А могут и [не быть](https://www.phoronix.com/scan.php?page=news_item&px=Qt-Wayland-NVIDIA-Thread), лучше проверьте работу своей системы c Plasma сперва с Wayland.

Выберете полную или минимальную установку этих [DE](https://wiki.archlinux.org/title/desktop_environment).  

Если вы еще не очень разбираетесь - ставьте полные группы пакетов и удаляйте ненужное. Такой подход уменьшит количество вероятных ошибок в работе DE. В противном случае ставьте минимальную версию и добавляйте нужные вам пакеты из полных групп.  

- #### Установим Gnome  

GNOME использует [Wayland](https://wiki.archlinux.org/title/wayland) по умолчанию.  

Минимальная установка:
```
sudo pacman -S gnome-shell gnome-terminal gnome-tweak-tool gnome-control-center xdg-user-dirs gdm gnome-keyring nautilus eog file-roller
```
Полная установка:
```
sudo pacman -S gnome gnome-extra
```

---  

Включим GPM и перезапустим систему:  

```
sudo systemctl enable gdm
```
```
reboot
```

Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|:--|
|`gnome-shell`|десктоп Gnome|
|`gnome-terminal`|терминал|
|`gnome-tweak-tool`|настройки для Gnome|
|`gnome-control-center`|настройки для рабочего стола Gnome|
|`xdg-user-dirs`|[менеджер пользовательских каталогов](https://wiki.archlinux.org/title/XDG_user_directories)|
|`gdm`|[менеджер дисплея Gnome](https://wiki.archlinux.org/title/GDM)|
|`gnome-keyring`|[хранитель паролей](https://wiki.archlinux.org/title/GNOME/Keyring)|
|`nautilus`|файловый менеджер|
|`eog`|просмотр фото|
|`file-roller`|архиватор|
|`gnome`|[группа пакетов](https://archlinux.org/groups/x86_64/gnome/) с десктопом и основными приложениями|
|`gnome-extra`|[группа пакетов](https://archlinux.org/groups/x86_64/gnome-extra/) с дополнительными приложениями|

---  

- #### Установим KDE Plasma

KDE установим с Xorg и поддержкой Wayland сессий  

Минимальная установка:
```
sudo pacman -S xorg-server xorg-apps plasma-wayland-session plasma-desktop sddm plasma-nm plasma-pa dolphin konsole kdeplasma-addons kde-gtk-config
```
Обычная установка:
```
sudo pacman -S xorg-server xorg-apps plasma-wayland-session plasma kde-applications
```
Список установленных пакетов:  

| Пакет   | Описание |
|:-----------|--|
|`xorg-server`|Xorg сервер|
|`xorg-apps`|[группа пакетов](https://archlinux.org/groups/x86_64/xorg-apps/) с конфигами для X|
|`plasma-wayland-session`|Wayland сессия Plasma|
|`plasma-desktop`|десктоп Plasma|
|`sddm`|[менеджер дисплея KDE](https://wiki.archlinux.org/title/SDDM)|
|`plasma-nm`|апплет Plasma для NetworkManager|
|`plasma-pa`|апплет Plasma для PulseAudio|
|`dolphin`|файловый менеджер|
|`konsole`|терминал|
|`kdeplasma-addons`|улучшения для Plasma|
|`kde-gtk-config`|интеграция с GTK приложениями|
|`plasma`|[группа пакетов](https://archlinux.org/groups/x86_64/plasma/) с десктопом и основными приложениями|
|`kde-applications`|[группа пакетов](https://archlinux.org/groups/x86_64/kde-applications/) с группами дополнительных приложениями(350 пакетов!)|

Вместо kde-application можно выбрать только нужные вам группы [тут](https://archlinux.org/packages/extra/any/kde-applications-meta/) или [тут](https://archlinux.org/packages/kde-unstable/any/kde-applications-meta/)


---  

## Начальная настройка  

- #### Настроим драйвер NVIDIA  

Сперва сгенерируем файл конфигурации X  

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


- #### Установим zsh и powerlevel10k  

```
sudo pacman -S zsh
```
```
sudo usermod -s /bin/zsh sonic
```
Перезапустим терминал. Выберем нужные нам опции в мастере настройки `zsh`
```
yay zsh-theme-powerlevel10k-git
```
```
echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```
```
exec zsh
```
Выберем нужные нам опции в мастере настройки `p10k`. Запустить вручную его можно с помощью команды `p10k configure`.  

---

Сейчас нашей системой можно пользоваться. Нужные приложения можно устанавливать из AUR с помощью `yay`.  

В следующей статье я опишу оптимизацию производительности и составлю список используемых мною пакетов.  

Если у вас есть какие-то вопросы по теме, то я с удовольствием отвечу на них в комментариях. Рад, если эта статья была вам полезна!

