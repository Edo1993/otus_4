# otus_4
Загрузка системы

   Задание:
    
     - Попасть в систему без пароля несколькими способами
     - Установить систему с LVM, после чего переименовать VG
     - Добавить модуль в initrd
     
1. Попасть в систему без пароля несколькими способами

Для получения доступа открываем GUI VirtualBox, запускаем вм. При выборе ядра для загрузки нажать e - в данном контексте edit.
![Image alt](https://github.com/Edo1993/otus_4/raw/master/1.png)
Попадаем в окно где мы можем изменить параметры загрузки
![Image alt](https://github.com/Edo1993/otus_4/raw/master/2.png)

  1.1 ```rw init=/sysroot/bin/sh```
  
В строке начинающейся с linux16 заменяем ```ro``` на ```rw init=/sysroot/bin/sh```, удаляем параметр ```console=ttS0,115200n8``` и нажимаем сtrl+x для загрузки в систему.
![Image alt](https://github.com/Edo1993/otus_4/raw/master/sysr1.png)
Файловая система сразу смонтирована в режим Read-Write
![Image alt](https://github.com/Edo1993/otus_4/raw/master/sysr2.png)
![Image alt](https://github.com/Edo1993/otus_4/raw/master/sysr3.png)
  1.2 ```rd.break```
  
В конце строки начинающейся с linux16 добавляем ```rd.break console=tty1``` и нажимаем сtrl-x для загрузки в систему.
![Image alt](https://github.com/Edo1993/otus_4/raw/master/2.png)
Попадаем в emergency mode. Корневая файловая система смонтирована. Попасть в нее и поменять пароль администратора:
![Image alt](https://github.com/Edo1993/otus_4/raw/master/3.png)
```
mount -o remount,rw /sysroot
chroot /sysroot
passwd
touch /.autorelabel
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/4.png)
После чего можно перезагружаться и заходить в систему с новым паролем.

  1.3 ```init=/bin/sh```
  
В конце строки начинающейся с linux16 удаляем параметр ```console=ttS0,115200n8```, добавляем ```init=/bin/sh```,
нажимаем сtrl-x для загрузки в систему
![Image alt](https://github.com/Edo1993/otus_4/raw/master/s1.png)
Рутовая файловая система при этом монтируется в режиме Read-Only.
![Image alt](https://github.com/Edo1993/otus_4/raw/master/s2.png)
Если вы хотите перемонтировать ее в режим Read-Write можно воспользоваться командой:
mount -o remount,rw /
После чего можно убедиться записав данные в любой файл или прочитав вывод команды:
mount | grep root
![Image alt](https://github.com/Edo1993/otus_4/raw/master/s3.png)


2. Установить систему с LVM, после чего переименовать VG

Пункты 2 и 3 были записаны с помощью утилиты script, можно проверить [тут](https://gist.github.com/lalbrekht/e51b2580b47bb5a150bd1a002f16ae85)

Первым делом посмотрим текущее состояние системы (нас интересует вторая строка с именем Volume Group):
```
vgs
```
Приступим к переименованию:
```
vgrename VolGroup00 OtusRoot
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/21.png)

Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg.
```
vim /etc/fstab
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/22.png)
```
vim /etc/default/grub
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/23.png)
Для правки /boot/grub2/grub.cfg используем sed
```
cat /boot/grub2/grub.cfg | grep VolGroup00
sed -i 's/VolGroup00/OtusRoot/g' /boot/grub2/grub.cfg
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/24.png)
Пересоздаем initrd image, чтобы он знал новое название Volume Group:
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/26.png)
После чего можем перезагружаемся с новым именем Volume Group и проверяем
```
vgs
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/25.png)


3. Добавить модуль в initrd
В каталоге с скриптами модулей создаем папку с именем 01test:
```
mkdir /usr/lib/dracut/modules.d/01test
```
В нее помещаем два скрипта:

1.[module-setup.sh](https://gist.github.com/lalbrekht/e51b2580b47bb5a150bd1a002f16ae85) - который устанавливает модуль и вызывает скрипт test.sh

2.[test.sh](https://gist.github.com/lalbrekht/ac45d7a6c6856baea348e64fac43faf0) - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик

Пересобираем образ initrd
```
dracut -f -v
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/31.png)
Можно проверить/посмотреть какие модули загружены в образ:
```
lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
```
![Image alt](https://github.com/Edo1993/otus_4/raw/master/32.png)
После чего можно пойти двумя путями для проверки:○Перезагрузиться и руками выключить опции  и увидеть вывод○Либо Отредактируем grub.cfg убрав эти опции:```rghb, quiet```
Для редактирования
```
vim /etc/default/grub
grub2-mkconfig -o /boot/grub2/grub.cfg
```

В итоге при загрузке будет пауза на 10 секунд и пингвин в выводе терминала ^__^
![Image alt](https://github.com/Edo1993/otus_4/raw/master/33.png)
