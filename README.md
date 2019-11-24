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
