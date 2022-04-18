---
title: Улучшение и оптимизация Arch
date: '2022-04-18'
tags: ['arch', 'zsh']
draft: true
summary: Улучшения, оптимизация работы и немного украшательств.
images: []
---

`Статья в разработке`

<TOCInline toc={props.toc} asDisclosure />

---

Привет! В этой статье мы настроим и оптимизируем наш Arch. 

## Установка и настройка zsh  

---

- #### Установим zsh:    

[zsh](https://wiki.archlinux.org/title/Zsh) - современный эмулятор терминала. Улучшенное дополнение команд и куча разных улучшений.

```
yay -S zsh
```

---

- #### Сменим оболочку на zsh:  

```
chsh -s /bin/zsh
```

---

- #### Запустим zsh и выполним базовую настройку с помощью мастера:

```
zsh
```  

Я увеличил количество строк в $HISTFILE и в редакторе строк сменил хоткеи на vi.  

---

- #### Установим oh-my-zsh из git:  

[oh-my-zsh](https://ohmyz.sh/) - платформа для управления конфигурацией `zsh`. Удобно управлять плагинами и темами zsh.  

```
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Для обновления время от времени запускайте `update_oh_my_zsh`  

---

- #### Установим плагины zsh:

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

В файле `~/.zshrc` добавим плагины `archlinux`, `zsh-syntax-highlighting` и `zsh-autosuggestions`:

```
73 plugins=(git archlinux zsh-syntax-highlighting zsh-autosuggestions)
```

Обновим `~/.zshrc` и перезапустим терминал

```
source .zshrc
```

Плагин `archlinux` добавляет алиасы для `pacman` и разных помощников aur. Список можно посмотреть [тут](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/archlinux/archlinux.plugin.zsh).  
Плагин [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) включит подсветку синтаксиса команд в терминале. Уменьшит количество ошибок и увеличит читаемость длинных команд.  
Плагин [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) добавляет предложения команд по мере ввода на основе истории.  

---

- #### Установим powerlevel10k:  

[powerlevel10k](https://github.com/romkatv/powerlevel10k) - тема для `zsh` (кросивое). 

```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Установим шрифт Meslo

```
yay ttf-meslo-nerd-font-powerlevel10k
```  

[Тут](https://github.com/romkatv/powerlevel10k/blob/master/font.md#recommended-font-meslo-nerd-font-patched-for-powerlevel10k) инструкция, как установить его в вашем эмуляторе терминала.  

В файле `~/.zshrc` установим тему 'powerlevel10k':

```
11 ZSH_THEME="powerlevel10k/powerlevel10k"
```

Обновим `~/.zshrc` и перезапустим терминал: 

```
source .zshrc
```  

Запустится мастер настройки темы p10k. Настроим внешний вид терминала так, как нравится нам. Чтобы запустить мастер снова - выполните `p10k configure`.

---

- #### Установим тему терминала через скрипт Gogh  

```
sh -c "$(wget -qO- https://git.io/vQgMr)"
```

[Gogh](https://mayccoll.github.io/Gogh/) - набор палитр для разных терминалов.

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