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
