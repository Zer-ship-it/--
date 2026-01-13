# --
Техническая документация по подключению Alt Workstation и Alt Jeos

z1 BR SRV

apt-get update

apt-get install task-samba-dc alterator-{fbi,net-domain} admx-* admc gpui -y

systemctl enable --now ahttpd alteratord 


domainname au-team.irpo

mcedit /etc/sysconfig/network (должен быть прописан hostname) F10

rm -rf /etc/samba/smb.conf /var/{lib,cache}/samba 

mkdir -p /var/lib/samba/sysvol 

samba-tool domain provision --realm=au-team.irpo --domain=au-team --adminpass='P@ssw0rd' --dns-backend=BIND9_DLZ --server-role=dc --use-rcf2307

z2z3 HQ-SRV

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

HQ-CLI

mkdir /mnt/nfs

chmod 777 /mnt/nfs

mcedit /etc/fstab

добавляем

192.168.1.2:/raid /nfs /mnt/nfs		nfs	defaults	

mount –a

df -h

z4 ISP

apt-get install –y chrony

vim /etc/chrony.conf local stratum 0

allow 172.16.4.0/28

allow 172.16.5.0/28

systemctl restart chronyd

systemctl restart chronyd

systemctl enable –now chronyd

HQ-RTR 

mcedit /etc/chrony.conf 

pool 172.16.4.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

BR-RTR 

mcedit /etc/chrony.conf 

pool 172.16.5.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

HQ-SRV

mcedit /etc/chrony.conf 

pool 172.16.4.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

BR-SRV

mcedit /etc/chrony.conf 

pool 172.16.5.1 iburst

systemctl restart chronyd

systemctl enable –now chronyd

chronyc sources

HQ-CLI

mcedit /etc/chrony.conf 

pool 172.16.4.1 iburst

systemctl restart chronyd.service

systemctl enable –now chronyd

chronyc sources

ISP

chronyc clients 

z5 BR-SRV

apt-get update && apt-get install ansible sshpass –y

 если такая проблема переходим в 
 
cd /etc/apt/sources.list.d/

mcedit alt.list меняем с .org на .ru

rpm [p10] http://ftp.altlinux.ru/pub/distributions/ALTLinux p10/branch/x86_64 classic gostcrypto

rpm [p10] http://ftp.altlinux.ru/pub/distributions/ALTLinux p10/branch/x86_64-i586 classic

rpm [p10] http://ftp.altlinux.ru/pub/distributions/ALTLinux p10/branch/noarch classic

apt-get update && apt-get install ansible sshpass –y

HQ-CLI

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

BR-SRV

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



