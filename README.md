# LDAP

## Установка OpenLDAP и веб интерфеса LAM
При установке будет запрошен пароль пользователя admin.
```
sudo apt install slapd ldap-utils
```
После установки пакетов надо переконфигурировать сервер, задать свои данные. Первое, что мы сделаем — создадим свой домен «mydomain.com». Для этого выполним команду
```
sudo dpkg-reconfigure slapd
```
1. Отвечаем No.
2. Введем данные для нашего домена. У нас это будет «mydomain.com». Нажимаем OK.
3. Вводим название организации «mydomain». Нажимаем OK.
4. Вводим пароль администратора LDAP-сервера. Пароль желательно вводить сложный, это  как-никак пароль администратора. Нажимаем ОК.
5. Повторяем
6. На вопрос удалять ли базу данных при удалении slapd отвечаем No
7. На вопрос о перемещении старой базы данных также отвечаем утвердительно

На этом первоначальная настройка завершена. 
Проверим, есть ли данные в базе:
```
sudo slapcat
```
Источник https://mnorin.com/ldap-ustanovka-i-nastrojka-ldap-servera.html

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

