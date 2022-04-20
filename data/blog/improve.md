---
title: Улучшение и оптимизация Arch
date: '2022-04-18'
tags: ['arch', 'zsh', 'customize']
draft: false
summary: Сделаем наш Arch удобнее, производительнее и красивее.
images: []
---

`Статья в разработке`

<TOCInline toc={props.toc} asDisclosure />

---

Привет! В этой статье мы улучшим работу в терминале и оптимизируем наш [десктоп на Arch](https://www.kittan.ru/blog/gui).  

Мы используем:  

- командный интерпретатор `zsh`  
- платформу для управления конфигурацией zsh `oh-my-zsh`  
- тему для zsh `powerlevel10k`  
- палитры для терминал `Gogh`  
- демон распределения ресурсов `Ananicy`  
- систему шины сообщений `dbus-broker`  

---

## Улучшение эмулятора терминала

- #### Установим zsh    

```
yay -S zsh
```  

[zsh](https://wiki.archlinux.org/title/Zsh) - современный командный интерпретатор. Улучшенное дополнение команд и куча разных улучшений.  

---

- #### Сменим оболочку на zsh  

```
chsh -s /bin/zsh
```

---

- #### Запустим zsh

```
zsh
```  

После запуска появится мастер настройки. Я увеличил количество строк в $HISTFILE и в редакторе строк сменил хоткеи на vi. Изменить настройки можно в файле `~/.zshrc`.  

---

- #### Установим oh-my-zsh  

```
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```  

[oh-my-zsh](https://ohmyz.sh/) - платформа для управления конфигурацией `zsh`. Удобно управлять плагинами и темами zsh. Для обновления иногда запускайте `update_oh_my_zsh`.  

---

- #### Установим плагины zsh

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

- В файле `~/.zshrc` добавим плагины `archlinux`, `zsh-syntax-highlighting` и `zsh-autosuggestions`:

```bash
73 plugins=(git archlinux zsh-syntax-highlighting zsh-autosuggestions)
```

- Обновим `~/.zshrc` и перезапустим терминал:

```
source .zshrc
```

Плагин `archlinux` добавляет алиасы для `pacman` и разных помощников aur. Список можно посмотреть [тут](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/archlinux/archlinux.plugin.zsh).  
Плагин [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) включит подсветку синтаксиса команд в терминале. Уменьшит количество ошибок и увеличит читаемость длинных команд.  
Плагин [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) добавляет предложения команд по мере ввода на основе истории.  

---

- #### Установим powerlevel10k  

```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
[powerlevel10k](https://github.com/romkatv/powerlevel10k) - тема для `zsh` (кросивое).  

- Установим шрифт Meslo:  

```
yay ttf-meslo-nerd-font-powerlevel10k
```  

[Тут](https://github.com/romkatv/powerlevel10k/blob/master/font.md#recommended-font-meslo-nerd-font-patched-for-powerlevel10k) инструкция, как установить его в вашем эмуляторе терминала.  

- В файле `~/.zshrc` установим тему 'powerlevel10k':

```bash
11 ZSH_THEME="powerlevel10k/powerlevel10k"
```

- Обновим `~/.zshrc` и перезапустим терминал: 

```
source .zshrc
```  

Запустится мастер настройки темы p10k. Настроим внешний вид терминала так, как нравится нам. Чтобы запустить мастер снова - выполните `p10k configure`.

---

- #### Установим палитру терминала  

[Gogh](https://mayccoll.github.io/Gogh/) - набор палитр для разных терминалов. Запустим скрипт и выберем понравившуюся нам палитру:  

```
sh -c "$(wget -qO- https://git.io/vQgMr)"
```

---

## Улучшение рабочего стола

- #### Добавим расширения GNOME

В GNOME расширения распространяются через [этот сайт](https://extensions.gnome.org/). Чтобы пользоваться ими нужно установить расширение для браузера и коннектор.  

Установим коннектор:  

```
yay chrome-gnome-shell
```

И установим [это](https://chrome.google.com/webstore/detail/gnome-shell-integration/gphhapmejobijbbhgpjhcjognlahblep) расширение для google-chrome. Теперь мы можем устанавливать расширения для GNOME!  

Список наших расширений:
| Расширение   | Описание |
|:-----------|:--|
|[BaBar Task Bar](https://extensions.gnome.org/extension/4000/babar/)|панель задач|
|[Big Sur Status Area](https://extensions.gnome.org/extension/4085/big-sur-status-area/)|переносит пункты системного трея из выпадающего меню в область уведомлений|
|[Sound Input & Output Device Chooser ](https://extensions.gnome.org/extension/906/sound-output-device-chooser/)|быстрая смена устройств ввода/вывода звука|
|[Tray Icons: Reloaded](https://extensions.gnome.org/extension/2890/tray-icons-reloaded/)|трей дополнительных приложений на верхней панели|

---

## Оптимизация производительности

- #### Установим улучшения драйвера NVIDIA

[nvidia-tweaks](https://aur.archlinux.org/packages/nvidia-tweaks) - сборник улучшений драйвера NVIDIA  

```
yay -S nvidia-tweaks
```

---

- #### Установим Ananicy

[Ananicy](https://github.com/Nefelim4ag/Ananicy) - демон распределения ресурсов для популярных приложений. Поможет избежать лагов и фризов.  

```
yay -S ananicy-git
```
```
sudo systemctl enable --now ananicy
```

---

- #### Включим TRIM диска

[TRIM](https://en.wikipedia.org/wiki/Trim_(computing)) - команда, сообщающая SSD о незадействованных блоках. Запись в пустые блоки происходит быстрее, что ускоряет работу системы.   

Запустим таймер 'TRIM'  

```
sudo systemctl enable fstrim.timer
```

---

- #### Установим dbus-broker

[dbus-broker](https://wiki.archlinux.org/title/D-Bus#dbus-broker) - оптимизированная система шины сообщений. Нужна для взаимодействия между процессами. Заменяет собой `dbus`.  

Установим `dbus-broker`  

```
sudo pacman -S dbus-broker
```

Отключим `dbus.service` и запустим `dbus-broker.service`  

```
sudo systemctl disable dbus.service
```
```
sudo systemctl enable --now dbus-broker.service
```

---

- #### Добавим диск в fstab

У меня есть старый `NTFS` hdd, который я использую как помоечку. Чтобы каждый раз не подключать его руками добавим запись о нём в /etc/fstab.  

В прошлой статье мы создали `fstab` [основанный на UUID](https://www.kittan.ru/blog/archinstall#%D0%B7%D0%B0%D0%BF%D0%B8%D1%88%D0%B5%D0%BC-%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D1%8E-%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85-%D0%BD%D0%B0%D0%BC%D0%B8-%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%B2%D1%8B%D1%85-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%B0%D1%85-%D0%B2-etcfstab). Поэтому узнаем `UUID` нашего диска:  

```
lsblk -o NAME,SIZE,UUID
```

В корневом каталоге создадим папку, в которую примонтируем диск:

```
sudo mkdir /data
```

В файле `/etc/fstab` создадим следующую запись:

```bash
# /dev/sdXx
UUID=НАШ_UUID /data ntfs nofail,x-systemd.device-timeout=1ms,rw,utf8  0   0
```
Опция `nofail` исключит ошибки при загрузке при отключенном диске.  
Опция `x-systemd.device-timeout` установит задержку вместо 90 секунд при отключенном диске.  

---

Наконец наша система готова. [Вот так](https://www.reddit.com/r/unixporn/comments/u73i7f/gnome_arch/) выглядит то, что вышло у меня.  

Следующая статья будет про QEMU/KVM - open source гипервизор с efi и снапшотами! Он отлично подходит для всяких странных вещей :alien:  

Надеюсь, эта статья была полезна вам! А если у вас возникла проблема, вы можете рассказать о ней в комментариях. Я обязательно отвечу.
