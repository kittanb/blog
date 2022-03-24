---
title: Установка и настройка Xen
date: '2022-03-17'
tags: ['arch', 'KDE', 'linux']
draft: true
summary: 
images: []
---

grep -E "(vmx|svm)" --color=always /proc/cpuinfo

yay xen

sudo pacman -S edk2-ovmf (seabios)