# redmine_docker
ИНСТРУКЦИЯ ПО ПОЛУЧЕНИЮ SSL СЕРТИФИКАТА И СМЕНЫ ПРОТОКОЛА С HTTP НА HTTPS ДЛЯ СЕРВЕРА С УЖЕ УСТАНОВЛЕННЫМ REDMINE
1. Копируем все новые yml файлы папку /root/
Это будет следующий перечень файлов:
certbot.yml
main.yml
nginx-fr.yml

2. Копируем все новые conf файлы папку /root/data/nginx (ПУСТЬ ЛЕЖАТ)
Это будет следующий перечень файлов:
redmine-fr.conf
redmine.conf

3. Копируем все новые файлы ТЕМЫ в папку /root/
 git clone https://github.com/yenihayat/redmine-theme-yh.git

(!!!ПУТЬ СТРОГО ТАКОЙ /root/redmine-theme-yh - и далее в ней лежат все файлы и подпапки из списка ниже!!!)
Это будет папка с названием "redmine-theme-yh" в ней содержатся папки и файлы:
.git
favicon
images
javascripts
screenshots
stylesheets
.gitattributes
.gitignore
LICENSE
README.md

4. Загружаем образы необходимых контейнеров:
docker pull nginx
docker pull certbot/certbot
		НЕ ЗАПУСКАЕМ образы после загрузки!

5. создаём директории для хранилищ сертификатов и конфига сайта nginx:
mkdir -p /root/data/nginx
mkdir -p /root/data/letsencrypt

6. Заходим в файл /root/data/nginx/redmine-fr.conf и создаём следующие настройки для конкретного сервера:
server_name "example.ru";

7. Заходим в файл /root/data/nginx/redmine.conf и создаём следующие настройки для конкретного сервера:
server_name "example.ru"; 


8. Правим переменнные команды в файле certbot.yml на правильные для конкретного сервера:
-d "example.ru";

9. останавливаем существующие контейнеры веб-приложения и БД (ТОЛЬКО ЕСЛИ УБЕДИЛИСЬ В КОРРЕКТНОСТИ ШАГА "-1")
docker-compose -f /opt/beget/redmine/docker-compose.yml down
		Ждем и проверяем командой docker ps, что ничего не запущено
		Проверяем что сайт НЕ РАБОТАЕТ!!! (Мало ли он запущен из другого места)

10. поднимаем контейнер с nginx
docker-compose -f nginx-fr.yml up
		контейнер начнёт сыпать на экран в stdout логи - пусть.

11. поднимаем контейнер с certbot в соседней консоли. контейнер с nginx при этом работает!
cd /root/
docker-compose -f certbot.yml up
		если всё сделано верно, 
		то в stdout на экране вывалится лог выполненых certbot'ом действий, 
		и указанием куда сохранились сертификат и приватный ключ
		К примеру, если все было выполнено верно, то пути будут следующие:
		/etc/letsencrypt/live/example.ru/fullchain.pem;
		/etc/letsencrypt/live/example.ru/privkey.pem;

12. останавливаем контейнер с nginx по Ctrl+C в соответствующей консоли


13. Залить файл .env командой
cp /opt/beget/redmine/.env /root/.env

14. поднимаем связку контейнеров БД, веб-приложение, nginx
docker-compose -f main.yml up -d

15. Если новая тема не встала по умолчанию, то выполнить следующую настройку в интерфейсе:
Go to "Administration / Settings / Display" and choose this theme (redmine-theme-yh) in themes list.
Save your changes.
