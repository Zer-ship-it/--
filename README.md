# --
Техническая документация по подключению Alt Workstation и Alt Jeos

z1 B-S

apt-get update

apt-get install task-samba-dc alterator-{fbi,net-domain} admx-* admc gpui -y

systemctl enable --now ahttpd alteratord 


domainname au-team.irpo

mcedit /etc/sysconfig/network (должен быть прописан hostname) F10

rm -rf /etc/samba/smb.conf /var/{lib,cache}/samba 

mkdir -p /var/lib/samba/sysvol 

samba-tool domain provision --realm=au-team.irpo --domain=au-team --adminpass='P@ssw0rd' --dns-backend=BIND9_DLZ --server-role=dc 

H-C
apt-get update

apt-get install gnome3-default lightdm

Перезагружаем и заходим в Gnome

apt-get install admx-* admc gpui sudo gpupdate -y

192.168.3.10:8080 (центр управления системой)

авторизируемся

настройки

режим эксперта

Веб-интерфейс

Порт с 8080 на 8081

применить

перезагрузить HTTP сервер

авторизируемся

Домен

Тип ДОМЕНА: Active Directory

dns server 192.168.3.10

пароль администратора 2раза 

применить (при успешном применении служба OK данные изменятся)

ПЕРЕКЛЮЧАЕМ ДНС В ПРОВОДНОМ СОЕДИНЕНИИ

ПАРАМЕТРЫ IPv4

dns 192.168.3.10 

сохранить 

nmcli 

nmcli con mod "Проводное соединение 1" ipv4.ignore-auto-dns yes

nmcli con show

ipv4 dns 192.168.3.10 (должен быть)

acc

аутентификация 

Домен Active Directory

рабочая группа au-team

Administator

P@ssw0rd (если невозможно найти домен выкл/вкл поддержку сети - интерфейс)

B-S
samba-tool group add hq (создаём группу hq) 

for i in $(seq 1 5); do samba-tool user add user$i.hq 'P@ssw0rd'; done

for i in $(seq 1 5); do samba-tool group addmembers hq user$i.hq; done 

samba-tool group list

samba-tool group listmembers hq 

admx-msi-setup

H-C

admx-msi-setup

roleadd hq wheel

rolelst

mcedit /etc/sudoers

находим User_Alias WHEEL_USERS = %wheel,  добавляем %AU-TEAM\\hq

находим Cmnd_Alias и пишем в конце

Cmnd_Alias SHELLCMD = /usr/bin/id, /bin/cat, /bin/grep

находим # WHEEL_USERS ALL=(ALL:ALL) ALL и изменяем на WHEEL_USERS ALL=(ALL:ALL) SHELLCMD (# УДАЛЯЕМ)

СОХРАНЯЕМ

exit

kinit

admc 

Объекты групповой политики > au-team.irpo > правой кнопкой > создать групповую политику > имя *sudoers*> принудительно галочка

> правой кнопокой > edit > ПК >Административные шаблоны>Samba>Настройки Unix>Управление разпешениями Sudo>Включено
> 
>Редактирова>(Добавляем команды которые разрешены /usr/bin/id, /bin/cat, /bin/grep)>ОК>gpupdate -f
Заходим в пользователя из группы hq
> 
user3.hq

P@ssw0rd

sudo id (проверям работают команды или нет)


z2z3 H-S

lsbik

mdadm –C /dev/md0 –l 0 –n 2 /dev/sd{b,c}

lsbik

mkfs.ext4 /dev/md0

echo DEVICE partitions >> /etc/mdadm.conf

mdadm –detail –scan >> /etc/mdadm.conf

mkdir /raid

mcedit /etc/fstab

/dev/md0	/raid ext4   defaults	0	0

mount –a

df –h

apt-get install –y nfs-{server,utils}

mkdir /raid/nfs

chmod 766 /raid/nfs

mcedit /etc/exports 

первая строка закомментируем # 
добавляем

/raid  /nfs 192.168.1.64/28(rw,no_subtree_check,no_root_squash)

exportfs –arv

systemctl enable –now nfs-server.service

H-C

mkdir /mnt/nfs

chmod 777 /mnt/nfs

mcedit /etc/fstab

добавляем

192.168.1.2:/raid /nfs /mnt/nfs		nfs	defaults	

mount –a

df -h

z4 I

apt-get install –y chrony

vim /etc/chrony.conf local stratum 0

allow 172.16.4.0/28

allow 172.16.5.0/28

systemctl restart chronyd

systemctl restart chronyd

systemctl enable –now chronyd

H-R 

mcedit /etc/chrony.conf 

pool 172.16.4.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

B-R 

mcedit /etc/chrony.conf 

pool 172.16.5.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

H-S

mcedit /etc/chrony.conf 

pool 172.16.4.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

B-S

mcedit /etc/chrony.conf 

pool 172.16.5.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

H-C

mcedit /etc/chrony.conf 

pool 172.16.4.1 iburst

systemctl restart chronyd.service

systemctl enable –now chronyd

chronyc sources

I

chronyc clients 

z5 B-S

apt-get update && apt-get install ansible sshpass –y

 если такая проблема переходим в 
 
cd /etc/apt/sources.list.d/

mcedit alt.list меняем с .org на .ru

rpm [p10] http://ftp.altlinux.ru/pub/distributions/ALTLinux p10/branch/x86_64 classic gostcrypto

rpm [p10] http://ftp.altlinux.ru/pub/distributions/ALTLinux p10/branch/x86_64-i586 classic

rpm [p10] http://ftp.altlinux.ru/pub/distributions/ALTLinux p10/branch/noarch classic

apt-get update && apt-get install ansible sshpass –y

H-C

cd /etc/openssh/

mcedit sshd_config 

раскоментируем Port 2026

раскоментируем MaxAuthTries и меняем на 2

раскоментируем PubkeyAuthentication yes

раскоментируем PasswordAuthentication yes

systecmtl restart sshd

mcedit sshd_config

AllowUsers sysadmin

systecmtl restart sshd

systecmtl enable sshd

systecmtl status sshd

B-S

cd /etc/ansible/

mcedit hosts 

[Alt]

192.168.1.1

192.168.2.1

[Alt:vars]

ansible_ssh_user=admin

ansible_ssh_pass=admin

ansible_connection=network_cli

ansible_network_os=ios

[Alt]

192.168.1.2 ansible_ssh_user=sshuser ansible_ssh_pass=P@ssw0rd

192.168.1.66 ansible_ssh_user=sysadmin ansible_ssh_pass=toor

[Alt:vars]

ansible_port=2024

ansible –m ping all

mcedit ansible.cfg 

[defaults]

interpreter_python = /usr/bin/python3

host_key_checking = False раскоментируем 

ansible –m ping all



