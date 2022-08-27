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
