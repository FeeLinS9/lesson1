### Обновления ядра и создание образа системы

Описание домашнего задания
1) Обновить ядро ОС из репозитория ELRepo
2) Создать Vagrant box c помощью Packer
3) Загрузить Vagrant box в Vagrant Cloud

Создадим Vagrantfile с содержимым из методички.  

После создания Vagrantfile, запустим виртуальную машину командой `vagrant up`  
Подключаемся к виртуальной машине `vagrant ssh`  
Проверим текущую версию ядра:  
```
[vagrant@kernel-update ~]$ uname -r
4.18.0-516.el8.x86_64
```

Подключим репозиторий, откуда возьмём необходимую версию ядра:
```
sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm 
```

Установим последнее ядро:
```
sudo yum --enablerepo elrepo-kernel install kernel-ml -y
```

Обновляем конфигурацию загрузчика:
```
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
Выбираем загрузку нового ядра по-умолчанию:
```
sudo grub2-set-default 0
```

Проверяем версию ядра:
```
[vagrant@kernel-update ~]$ uname -r
6.5.9-1.el8.elrepo.x86_64
```

На этом обновление ядра закончено.


### Создание образа системы

В каталоге с нашим Vagrantfile создадим папку packer `mkdir packer`, переходим в неё и создаём файл centos.json с нужным содержимым, папки `mkdir http`, `mkdir scripts`.  
В папке http создадим файл автоматической конфигурации ОС vagrant.ks, в папке scripts файлы stage-1-kernel-update.sh и stege-2-clean.sh.  
После того как мы создадим нужные файлы, переходим в папку packer `cd ..` и создаём образ системы `packer build centos.json`  

В связи с медленным VPN пришлось в файле centos.json изменить параметр "ssh_timeout": "90m" и убрать параметр "--nat-localhostreachable1", "on", так как он не используется в версии VB 6.1.

Создадим Vagrantfile на основе образа centos8-kernel6: 
```
vagrant init centos8-kernel6
```

Запустим нашу ВМ: `vagrant up`
Подключимся к ней по SSH: `vagrant ssh`
Проверим версию ядра: 
```
[vagrant@otus-c8 ~]$ uname -r
6.5.10-1.el8.elrepo.x86_64
```

#### Далее публикуем наш образ: 
```
vagrant cloud publish --release feelins/centos8-kernel6 1.0 virtualbox centos-8-kernel-6-x86_64-Minimal.box
```
