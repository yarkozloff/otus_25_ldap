# LDAP
## Описание/Пошаговая инструкция выполнения домашнего задания:
- Установить FreeIPA;
- Написать Ansible playbook для конфигурации клиента;
- 3*. Настроить аутентификацию по SSH-ключам;
- 4**. Firewall должен быть включен на сервере и на клиенте.
- Формат сдачи ДЗ - vagrant + ansible

## Установка FreeIPA
Централизованное управление доступом:
- На основе LDAP работает и Microsoft Active Directory, которая является , по факту , корпоративным стандартом на текущий момент.
- В мире opensource есть несколько реализаций ldapкаталогов , например openldap или apache directory server.
- Есть и другие , например NIS (Network Information Service) он же Yellow Pages.

LDAP (Lightweight Directory Access Protocol) не является протоколом аутентификации или авторизации. Протокол для хранения и получения данных из каталога с иерархической структурой. Пример данных: записи о пользователях, группах, контактаная информация, место работы. Сложности для понимания:
- своеобразная терминология и сокращения,
- используется как компонент других систем 
LDAP функционирует на 389/tcp без SSL/TLS и 636/tcp с SSL/TLS.

Настройка сервера:
- Используется локальный бокс centos7
- Устанавливаются необходимые компоненты: nss, ipa-server, ipa-server-dns
- Конфигурируется firewalld
- Конфигурируется freeipa:
```
- name: Configure freeipa
    command: |
      ipa-server-install -U \
      -r YARKOZLOFF.LOCAL \
      -n yarkozloff.local \
      -p 12345678 \
      -a 12345678 \
      --hostname=ipa.yarkozloff.local \
      --ip-address=10.0.2.15 \
      --mkhomedir \
      --setup-dns \
      --no-forwarders \
      --no-reverse
```
Подключаемся к машине смотрим статус работы служб FreeIPA:
```
[root@ipa ~]# ipactl status
Directory Service: RUNNING
krb5kdc Service: RUNNING
kadmin Service: RUNNING
named Service: RUNNING
httpd Service: RUNNING
ipa-custodia Service: RUNNING
ntpd Service: RUNNING
pki-tomcatd Service: RUNNING
ipa-otpd Service: RUNNING
ipa-dnskeysyncd Service: RUNNING
ipa: INFO: The ipactl command was successful
```
Проверим, получит ли пользователь-администратор токен через Kerberos с помощью команды kinit, используя тот же пароль пользователя-администратора, который мы указали при установке FreeIPA.
```
[root@ipa ~]# kinit admin
Password for admin@YARKOZLOFF.LOCAL:
[root@ipa ~]# klist
Ticket cache: KEYRING:persistent:0:0
Default principal: admin@YARKOZLOFF.LOCAL

Valid starting       Expires              Service principal
08/27/2022 13:21:32  08/28/2022 13:21:28  krbtgt/YARKOZLOFF.LOCAL@YARKOZLOFF.LOCAL
```
## Ansible playbook для конфигурации клиента
Выбирая между хостовой машины или отдельной - мой выбор пал на отдельную клиентскую машину развернутую через vagrant. Для этого потребовался локальный бокс с centos7, установленный пакеты (некоторые потребовались для дебага) - bind-utils, ipa-client, realmd, ntp. Отключить selinux и firewalld. Также добавлена таска для автоматической конфигурации клиента:
```
  - name: Configure freeipa-client
    command: |
      ipa-client-install -U \
      --principal admin \
      --password 12345678 \
      --hostname=client.yarkozloff.local \
      --mkhomedir \
      --server=ipa.yarkozloff.local \
      --domain yarkozloff.local \
      --realm YARKOZLOFF.LOCAL \
      --force-ntpd \
```
При настройки клиента могут возникнуть ошибки из-за расхождения времени на сервере ipa и клиенте, поэтому был добавлен параметр --force-ntpd. После поднятия машины провижним её, подключаемся и проверяем получит ли пользователь-администратор токен через Kerberos с помощью команды kinit:
```
[root@client ~]# kinit admin
Password for admin@YARKOZLOFF.LOCAL:
[root@client ~]# klist
Ticket cache: KEYRING:persistent:0:0
Default principal: admin@YARKOZLOFF.LOCAL

Valid starting       Expires              Service principal
08/29/2022 20:59:01  08/30/2022 20:58:58  krbtgt/YARKOZLOFF.LOCAL@YARKOZLOFF.LOCAL
```
### Добавление пользователей
На сервере ipa создадим пользователя:
```
[root@ipa ~]# kinit admin
Password for admin@YARKOZLOFF.LOCAL:
[root@ipa ~]# ipa user-add yar --first=Ярослав --last=Козлов --password
Password:
Enter Password again to verify:
----------------
Added user "yar"
----------------
  User login: yar
  First name: Ярослав
  Last name: Козлов
  Full name: Ярослав Козлов
  Display name: Ярослав Козлов
  Initials: ЯК
  Home directory: /home/yar
  GECOS: Ярослав Козлов
  Login shell: /bin/sh
  Principal name: yar@YARKOZLOFF.LOCAL
  Principal alias: yar@YARKOZLOFF.LOCAL
  User password expiration: 20220829180217Z
  Email address: yar@yarkozloff.local
  UID: 1962000001
  GID: 1962000001
  Password: True
  Member of groups: ipausers
  Kerberos keys available: True
```
Получим тикет на клиенте:
```
[root@client ~]# kinit yar
Password for yar@YARKOZLOFF.LOCAL:
[root@client ~]# klist
Ticket cache: KEYRING:persistent:0:krb_ccache_97L6Frb
Default principal: yar@YARKOZLOFF.LOCAL

Valid starting       Expires              Service principal
08/29/2022 21:03:08  08/30/2022 21:03:05  krbtgt/YARKOZLOFF.LOCAL@YARKOZLOFF.LOCAL
```
