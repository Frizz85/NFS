# Домашнее задание "Vagrant стенд для NFS"

*Цели домашнего задания:*
Научиться самостоятельно развернуть сервис NFS и подключить к нему
клиента

## Описание/Пошаговая инструкция выполнения домашнего задания:

* vagrant up должен поднимать 2 виртуалки: виртуальных машины (сервер NFS и клиента);
* на сервере NFS должна быть подготовлена и экспортирована директория;
* экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины;
* монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3 по протоколу UDP; ;
* firewall должен быть включен и настроен как на клиенте, так и на сервере;
Методичка Стенд Vagrant с NFS https://docs.google.com/document/d/1lW327eKqGJwGEXnu_XkXMvwgPkRjz3HWS0gx-tvDCR4/edit?usp=sharing

## Запуск
```
vagrant up
```

### Настройка сервера NFS

```
vagrant ssh nfss
su -i
```

Установка NFS
```
yum nstall nfs-utils
```

Включение firewall
```
systemctl enable firewall --now
systemctl status firewalld
```

Разрешаем в firewall доступ к сервисам NFS
```
irewall-cmd --add-service="nfs3" \
--add-service="rpc-bind" \
--add-service="mountd" \
--permanent
firewall-cmd --reload
```

Включение сервера NFS (для конфигурации NFSv3 over UDP он не требует
дополнительной настройки. С настройкой по умолчанию можно ознакомиться 
в файле /etc/nfs.conf)

```
systemctl enable nfs --now
```

Проверка наличия слушаемых портов 2049/udp, 2049/tcp, 20048/udp,
20048/tcp, 111/udp, 111/tcp (не все они будут использоваться далее, но
их наличие сигнализирует о том, что необходимые сервисы готовы
принимать внешние подключения)

```
ss -tnplu | grep -E '2049|20048|111'
```

![files](nfs_img/1.JPG)

Cоздаём и настраиваем директорию, которая будет экспортирована в будущем

```
mkdir -p /srv/share/upload
chown -R nfsnobody:nfsnobody /srv/share
chmod 0777 /srv/share/upload
```

Создание в файле /etc/exports структуру, которая позволит экспортировать ранее созданную директорию
```
cat << EOF > /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
EOF
```

Экспорт директории /srv/share
```
exportfs -r
```

Проверка экспорта:
```
exportfs -s
```

![files](nfs_img/2.JPG)

### Настройка клиента NFS

```
vagrant ssh nfsc
su -l
```

Установка вспомогательных утилит:
```
yum install nfs-utils
```

Включение firewall
```
systemctl enable firewalld --now
systemctl status firewalld
```

Добавление в /etc/fstab строку
```
echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0" >> /etc/fstab
systemctl daemon-reload
systemctl restart remote-fs.target
```

Проверка успешности монтирования
```
mount | grep mnt
```

![files](nfs_img/3.JPG)

### Проверка работоспособности

Проверка созданного на сервере файла /srv/share/upload/check_file

![files](nfs_img/4.JPG)

Проверка созданного на клиенте файла /mnt/upload/client_file

![files](nfs_img/5.JPG)

Проверка работы сервера настроенного с помощью скрипта nfss.sh

![files](nfs_img/7.JPG)

Проверка работы клиента настроенного с помощью скрипта nfsc.sh

![files](nfs_img/8.JPG)

Проверка отображения на сервере файла test_check3 созданного на клиенте

![files](nfs_img/9.JPG)
