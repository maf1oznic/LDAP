# LDAP

## Установка OpenLDAP и веб интерфеса LAM
Установка OpenLDAP и утилит LDAP:
```
sudo apt install slapd ldap-utils
```
Процедура установки запросит пароль админа. На данный момент его можно оставить пустым, поскольку мы немедленно перенастроим slapd установку.
```
sudo dpkg-reconfigure slapd
```
Для каждого экрана сценария реконфигурации введите следующее:
1. НЕТ
2. example.com
3. example.com
4. Пароль
5. Повтор пароля
6. НЕТ
7. ДА

Проверим, есть ли данные в базе:
```
sudo slapcat
```
Импорт схемы ssh ключа:
```
nano openssh-lpk.ldif
```
```
# Add schema for OpenSSH Public Keys to LDAP Server
# Use with ~$ sudo ldapadd -Q -Y EXTERNAL -H ldapi:/// -f 10-openssh-lpk.ldif
#
dn: cn=openssh-lpk,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: openssh-lpk
olcAttributeTypes: ( 1.3.6.1.4.1.24552.500.1.1.1.13 NAME 'sshPublicKey'
  DESC 'MANDATORY: OpenSSH Public key'
  EQUALITY octetStringMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.40 )
olcObjectClasses: ( 1.3.6.1.4.1.24552.500.1.1.2.0 NAME 'ldapPublicKey' SUP top AUXILIARY
  DESC 'MANDATORY: OpenSSH LPK objectclass'
  MAY ( sshPublicKey $ uid )
  )
```
```
sudo ldapadd -Q -Y EXTERNAL -H ldapi:/// -f openssh-lpk.ldif
```
Установка зависимостей и веба
```
sudo apt install apache2 php php-cgi libapache2-mod-php php-mbstring php-common php-pear -y
sudo apt install ldap-account-manager -y
sudo a2enconf php*-cgi
sudo systemctl restart apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

## Настройка веб интерфеса
Заходим на http://domain/lam. В правом верхнем углу заходим в LAM configuration -> Edit general settings (пароль для входа: lam), в самом низу меняем пароль и сохраняем. Далее LAM configuration -> Edit server profiles (пароль для входа: lam), в Server settings -> List of valid users меняем Manager на admin и свой домен, в Tool settings указываем домен и внизу задаем пароль сохраняем. Переходим во вкладку Account types здесь также указываем свой домен и можно поменять People на Department чтобы делить пользователей на отделы и сохраняем. Переходим во вкладку Modules и добавляем у пользователей ssh ключи и hosts, у групп добавляем hosts сохраняем. Далее логинимся с паролем от ldap и создаем записи
## Создание групп и пользователей
https://www.youtube.com/watch?v=yGJERaeZmKc

## Конфигурация клиента
Установка зависимостей:
```
sudo apt -y install sssd libpam-sss libnss-sss libsss-sudo sssd-tools
```
Автоматическое создание домашнего каталога пользователя LDAP при первом входе:
```
sudo pam-auth-update --enable mkhomedir
```
Удаление пакета. Мешает при подключении по ключу:
```
sudo apt-get remove ec2-instance-connect
```
Отредактируйте файл /etc/ssh/sshd_config и включите следующие настройки:
```
AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys
AuthorizedKeysCommandUser root
```
```
sudo systemctl restart ssh
```
Создайте файл /etc/sssd/sssd.conf (Нужно поменять данные для подключения к LDAP по примеру)
```
sssd]
config_file_version = 2
reconnection_retries = 3
domains = dam-ldap.oraclesiebelcrm.com
services = nss, pam, ssh, sudo

[nss]
filter_groups = root
filter_users = root ldap.bind
entry_cache_nowait_percentage = 75

[pam]
offline_credentials_expiration = 30

[domain/dam-ldap.oraclesiebelcrm.com]
enumerate = true
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
sudo_provider = ldap
ldap_uri = ldap://dam-ldap.oraclesiebelcrm.com:389
ldap_search_base = dc=dam-ldap,dc=oraclesiebelcrm,dc=com
ldap_default_bind_dn = cn=admin,dc=dam-ldap,dc=oraclesiebelcrm,dc=com
ldap_default_authtok = Qwerty78
ldap_user_search_base = ou=Department,dc=dam-ldap,dc=oraclesiebelcrm,dc=com
ldap_group_search_base = ou=Groups,dc=dam-ldap,dc=oraclesiebelcrm,dc=com
ldap_group_member = memberUid
ldap_user_ssh_public_key = sshPublicKey
access_provider = ldap
ldap_access_filter = (|(objectClass=posixAccount)(objectClass=inetOrgPerson))
ldap_id_mapping = False
cache_credentials = True
ldap_user_name = uid
ldap_group_name = cn
```
Установка соответсвующий разрешений:
```
sudo chmod 600 /etc/sssd/sssd.conf
```
```
sudo systemctl restart sssd
```
При соответствующей настройке пользователи, настроенные на сервере LDAP, должны отображаться при выполнении команды getent passwd
