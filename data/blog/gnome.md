---
title: Подготовка Arch для работы
date: '2022-03-17'
tags: ['arch', 'gnome', 'linux']
draft: false
summary: 
images: []
---

<TOCInline toc={props.toc} asDisclosure />

Привет! 

- #### Включим 32-битный репозиторий  

Нам нужны будут некоторые зависимости оттуда.
Для этого в `/etc/pacman.conf` раскомментируем строки ниже
```
[multilib]
Include=/etc/pacman.d/mirrorlist
```
и сохраним файл.

- #### Обновим систему  
```
sudo pacman -Suy
```
- #### Установим yay  

[yay](https://github.com/Jguer/yay) - помощник AUR
Создадим каталог для git и перейдём в него. Я сделаю это в Download.  

```
mkdir /home/sonic/Download/git
cd Download/git/
```
Клонируем репозиторий с yay и установим его с помощью [makepkg](https://wiki.archlinux.org/title/Makepkg)  

```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
`-i` - установит пакет после сборки
`-s` - установит недостающие зависимости

- #### Установим X  
```
sudo pacman -S xorg-server xorg-apps
```
Список установленных пакетов  

| Пакет   | Описание |
|:-----------|--|
|`xorg-server`|X сервер|
|`xorg-apps`|[группа пакетов](https://archlinux.org/groups/x86_64/xorg-apps/) с конфигами для X. Тут 35 пакетов, ого|

- #### Установим драйвер NVIDIA и Vulkan
```
sudo pacman -S nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader opencl-nvidia lib32-opencl-nvidia libxnvctrl
```
Список установленных пакетов  

| Пакет   | Описание |
|:-----------|--|
|`nvidia-dkms`|[DKMS](https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support) проприетарный драйвер NVIDIA|
|`nvidia-utils`|утилиты драйвера NVIDIA|
|`lib32-nvidia-utils`|утилиты драйвера NVIDIA (32-bit)|
|`nvidia-settings`|[Редактор опций NVIDIA](https://wiki.archlinux.org/title/NVIDIA#nvidia-settings)|
|`vulkan-icd-loader`|[графический API Vulkan](https://wiki.archlinux.org/title/Vulkan)|
|`lib32-vulkan-icd-loader`|графический API Vulkan (32-bit)|
|`opencl-nvidia`|[среда выполнения OpenCL](https://wiki.archlinux.org/title/GPGPU#OpenCL) для NVIDIA|
|`lib32-opencl-nvidia`|среда выполнения OpenCL для NVIDIA (32-bit)|
|`libxnvctrl`|API для NVIDIA и X|


- #### Установим Gnome  

```
sudo pacman -S gnome-shell gnome-terminal gnome-tweak-tool gnome-control-center xdg-user-dirs gdm gnome-keyring nautilus eog file-roller
```
```
sudo systemctl enable gdm
```
```
reboot
```

Список установленных пакетов  

| Пакет   | Описание |
|:-----------|:--|
|`gnome-shell`|Gnome|
|`gnome-terminal`|терминал|
|`gnome-tweak-tool`|настройки для Gnome|
|`gnome-control-center`|настройки для рабочего стола Gnome|
|`xdg-user-dirs`|[менеджер пользовательских каталогов](https://wiki.archlinux.org/title/XDG_user_directories)|
|`gdm`|[менеджер дисплея Gnome](https://wiki.archlinux.org/title/GDM)|
|`gnome-keyring`|[хранитель паролей](https://wiki.archlinux.org/title/GNOME/Keyring)|
|`nautilus`|файловый менеджер|
|`eog`|просмотр фото|
|`file-roller`|архиватор|

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

Во вкладке `X Server XVideo Settings` выберем основной монитор
Во вкладке `PowerMizer` в разделе `PowerMizer Settings` выберем `Prefer Maximum Performance`
Во вкладке `X Server Display Configuration` выберем наше разрешение и частоту и сохраним `Save to X Configuration File`
Запустим `nvidia-settings` без `sudo` и повторим всё настройки выше. Но не будем сохранять через `Save to X Configuration File`

- #### Добавляем модули ядра для NVIDIA и brtfs

Отредактируем скрипт Initial ramdisk `/etc/mkinitcpio.conf`
В строку `MODULES` добавим `nvidia nvidia_modeset nvidia_uvm nvidia_drm crc32c libcrc32c zlib_deflate btrfs`
Теперь пересобираем образ ядра
```
sudo mkinitcpio -P
```
Обновляем загрузчик и перезагружаемся. Перекреститесь, если вы православный.
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
```
reboot
```