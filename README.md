# Kittygram
# Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Проект состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

## Технологии
Python 3.x
node.js 9.x.x
backend: Django
gunicorn
nginx
frontend: React
## Установка:


1. Склоинровать репозиторий
```
git clone git@github.com:Nikolay449/infra_sprint1.git
```
2. Создать и активировать виртуальное окружение
```
python3 -m venv venv
source venv/bin/activate
```
3. Установить зависимости для Python
```
pip install -r requirements.txt 
```
4. Перейти в папку backend и выполнить миграции
```
cd infra_sprint1/backend/
python manage.py migrate
```
5. Создать суперюзера
```
python manage.py createsuperuser
```
6. В файле `infra_sprint1/backend/kittygram_backend/settings.py` в переменную ALLOWED_HOSTS добавить локальные адреса и внешний IP, доменное имя.
```
ALLOWED_HOSTS = ['127.0.0.1', '0.0.0.0', 'localhost', 'xxx.xxx.xxx.xxx']
```
7. В этом же файле поменять значение переменной DEBUG с True на False
```
DEBUG = False
```
8. Установить node.js
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```
9. Перейти в `infra_sprint1/frontend` и установить зависимости для frontend-приложения
```
npm i
```
10. Для backend-приложения установить WSGI-сервер gunicorn
```
pip install gunicorn
```
11. Создать файл конфигурации для автозапуска WSGi-сервера
```
sudo nano /etc/systemd/system/gunicorn_kittygram.service
```
12. Внести в него следующие настройки
```
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=<имя-пользователя-в-системе>
WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8000 kittygram_backend.wsgi

[Install]
WantedBy=multi-user.target 
```
13. Запустить созданную службу и внести её в автозапуск
```
sudo systemctl start gunicorn_kittygram
sudo systemctl enable gunicorn_kittygram
```
14. Установить веб-сервер и запустить его
```
sudo apt install nginx -y
sudo systemctl start nginx 
```
15. Открыть порты для фаерволла и активировать его
```
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
```
16. Перейти в `infra_sprint1/frontend/`, скомпилировать frontend-приложение и копировать его в системную директорию веб-сервера
```
npm run build
sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/kittygram/
```
17. Перейти в `infra_sprint1/backend/`, собрать статику для админки Django и копировать его в системную директорию веб-сервера (Необходимо изменить название папки со статикой во избежание конфликта имён)

`infra_sprint1/backend/kittygram_backend/settings.py` поменять значение переменной STATIC_URL и добавить новую STATIC_ROOT
```
STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
python manage.py collectstatic
sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/
```
18. Создать папку для медиафайлов в директории веб-сервера, изменить права доступа
```
cd /var/www/kittygram/
mkdir media
sudo chown -R <имя_пользователя> /var/www/kittygram/media/
```
19. Перейти в файл конфигурации веб-сервера и изменить его настройки
```
sudo nano /etc/nginx/sites-enabled/default 
server {

    server_name server_name <публичный-IP-адрес> <доменное-имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }

    location / {
    root    /var/www/kittygram;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```
Проверить файл конфигурации веб-сервера, перезагрузить его конфиг в случае успеха
```
sudo nginx -t
sudo systemctl reload nginx
```
20. Получить SSL-сертификат для вашего доменного имени при помощи certbot
```
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
sudo certbot --nginx
```
