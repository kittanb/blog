---
title: Подготовка Arch для работы
date: '2022-03-17'
tags: ['arch', 'gnome', 'linux']
draft: false
summary: 
images: []
---

Привет! 

- #### Включим 32-битный репозиторий
Нам нужны будут некоторые зависимости оттуда.
Для этого в `/etc/pacman.conf` уберем решетки перед
```
#[multilib]
#Include=/etc/pacman.d/mirrorlist
```
и сохраним файл.

- #### Обновим систему
```
sudo pacman -Suy
```

- #### Установим X и драйвера NVIDIA
```
sudo pacman -S xorg-server xorg-apps nvidia-dkms nvidia-utils lib32-nvidia-utils nvidia-settings vulkan-icd-loader lib32-vulkan-icd-loader opencl-nvidia lib32-opencl-nvidia libxnvctrl
```
Список установленных пакетов

| Пакет   | Описание |
|:-----------|:--|
|`xorg-server`|X сервер|
|`xorg-apps`|[группа пакетов](https://archlinux.org/groups/x86_64/xorg-apps/) с конфигами для X. Тут 35 пакетов и кое-что можно будет удалить.|
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
