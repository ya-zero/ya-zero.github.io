---
title: "подписание rpm пакетов с помощью rutoken"
date: 2019-10-01 00:00:51 +0000
categories:
  - rpm 
  - rutoken
---

1) config.vm.box = "generic/ubuntu1904"

2) требуемые пакеты.
   2.1 dpkg -i libccid libpcsclite1 pcscd pcsc-tools opensc gnupg-pkcs11-scd  rpm
   2.2 wget https://download.rutoken.ru/Rutoken/PKCS11Lib/1.9.15.0/Linux/x64/librtpkcs11ecp_1.9.15.0-1_amd64.deb
           def install /usr/lib/librtpkcs11ecp.so

3) проверить есть ли пакеты 

3.1) dpkg -s gnupg gpg-agent / gnupg не ниже 2.1.9

3.2) dpkg -s gnupg-pkcs11-scd  / не ниже 0.9.2
  
4) конфигурим демоны

   gnupg-pkcs11-scd
   
   gpg-agent

в домашней дериктории пользователя 
```bash
./gnupg/gnupg-pkcs11-scd.conf
 debug-all
 verbose
 log-file /root/gnupg-pkcs11-scd.log
 providers p1
 provider-p1-library /usr/lib/librtpkcs11ecp.so 
```
```bash
.gnupg/gpg-agent.conf
 scdaemon-program /usr/bin/gnupg-pkcs11-scd
 pinentry-program /usr/bin/pinentry-tty
```
5)
```bash
 gnupg-pkcs11-scd --daemon
 gpg-agent  --daemon (по идее будет запущен)  можно  сделать gpgconf --kill all
 gpg-agent  --server
```
Для генерации opengpg ключа нужен KEY-FRIEDNLY  его можно взять из логов log-file /root/gnupg-pkcs11-scd.log 
```bash
gpg-connect-agent  'SCD LEARN' /bye | grep KEY-FRIEDNLY  | awk '{print $3}'
```

```bash
%echo Generating a OpenPGP key
 Key-Type: 1
 Key-Grip: 88E23DFBAA20FA2F8D42A2F62C24E409E8417662
 Key-usage: sign,cert
 Name-Real: name real
 Name-Comment: name comment
 Name-Email: test@mail.ru
 #Passphrase: 12345678
 %commit
 %done
```
что бы подписать нужен файл экпорт производиться по части описания Joe Tester (with stupid passphrase) <test@mail.ru>
gpg --export -a 'test@mail.ru' > RPM-GPG-KEY-faleman

rpm --import RPM-GPG-KEY-faleman


rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n'
   >>examples>>> gpg-pubkey-77b915ba-5d932ddb --> gpg(Joe Tester (with stupid passphrase) <test@mail.ru>)


/root/.rpmmacros
 %_gpg_name Joe Tester (with stupid passphrase) <sept_29@mail.ru>

rpmsign -addsign ./*rpm



p.s. 

vagrant файл и скрипты доступны по ссылке
https://github.com/ya-zero/ya-zero.github.io/tree/master/uploads/rutoken


