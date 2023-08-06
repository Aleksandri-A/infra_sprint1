# О проекте
 Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.
 
## Настройка бэкенд-приложения

Клонировать репозиторий и перейти в него в командной строке:

```
git clone git@github.com:Aleksandri-A/infra_sprint1.git
```

```
cd kittygram_backend
```

Cоздать и активировать виртуальное окружение:

```
python3 -m venv env
```

* Если у вас Linux/macOS

    ```
    source env/bin/activate
    ```

* Если у вас windows

    ```
    source env/scripts/activate
    ```

```
python3 -m pip install --upgrade pip
```

Установить зависимости из файла requirements.txt:

```
pip install -r requirements.txt
```

Выполнить миграции:

```
python3 manage.py migrate
```

Создайте суперпользователя:

```
python3 manage.py createsuperuser
```

В дирректории kittygram_backend создать файл .env и прописать ваши значения SECRET_KEY и ALLOWED_HOSTS в формате ключ = значение.

Соберите статику бэкенд-приложения:

```
python3 manage.py collectstatic
```

Скопируйте директорию static_backend/ в директорию /var/www/название_проекта/:

```
sudo cp -r путь_к_директории_с_бэкендом/static_backend /var/www/название_проекта
```

## Настройка фронтенд-приложения

 Находясь в директории с фронтенд-приложением, установите зависимости для него:
 
 ```
npm i
```

Из директории с фронтенд-приложением выполните команду:

 ```
npm run build
```

Скопируйте статику фронтенд-приложения в директорию по умолчанию:

 ```
sudo cp -r путь_к_директории_с_фронтенд-приложением/build/. /var/www/название_проекта/
```

## Установка и настройка WSGI-сервера Gunicorn

Перейдите в директорию с файлом manage.py, и запустите Gunicorn:

```
gunicorn --bind 0.0.0.0:8081 kittygram_backend.wsgi
```

Создайте файл конфигурации юнита systemd для Gunicorn в директории
/etc/systemd/system/. Назовите его gunicorn_название_проекта.service:

```
sudo nano /etc/systemd/system/gunicorn_название_проекта.service
```

Подставьте в код из листинга свои данные, добавьте этот код без комментариев в файл
конфигурации Gunicorn и сохраните изменения:

```
[Unit]
Description=gunicorn daemon
After=network.target
[Service]
User=имя_пользователя_в_системе
WorkingDirectory=/home/имя_пользователя/папка_с_проектом/папка_с_файлом_manage.py/
# Чтобы узнать путь до Gunicorn, воспользуйтесь командой which gunicorn.
ExecStart=/.../venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi:application
[Install]
WantedBy=multi-user.target
```

Команда sudo systemctl с параметрами start, stop или restart запустит, остановит
или перезапустит Gunicorn. Например, вот команда запуска:

```
sudo systemctl start gunicorn_название_проекта
```

Чтобы systemd следил за работой демона Gunicorn, запускал его при старте системы
и при необходимости перезапускал, используйте команду:

```
sudo systemctl enable gunicorn_название_проекта
```

## Установка и настройка веб- и прокси-сервера Nginx

Запустите Nginx командой:

```
sudo systemctl start nginx
```

Обновите настройки Nginx:

```
server {
 listen 80;
 server_name ваш_домен;
 location /api/ {
 proxy_pass http://127.0.0.1:8000;
 }

 location /admin/ {
 proxy_pass http://127.0.0.1:8000;
 }
 location / {
 root /var/www/имя_проекта;
 index index.html index.htm;
 try_files $uri /index.html;
 }
}
```

Сохраните изменения в файле, закройте его и проверьте на корректность:

```
sudo nginx -t
````

Перезагрузите конфигурацию Nginx:

```
sudo systemctl reload nginx
```

## Настройка файрвола ufw

Активируйте разрешение принимать запросы только на порты 80��443 и 22�

```
# 80, 443: с ними будут работать пользователи, делая запросы к приложению.
sudo ufw allow 'Nginx Full'
# 22: нужен, чтобы вы могли подключаться к серверу по SSH.
sudo ufw allow OpenSSH
```

Включите файрвол:

```
sudo ufw enable
```
>В терминале выведется запрос на подтверждение операции. Введите y и нажмите Enter.


Проверьте работу файрвола:

```
sudo ufw status
```

## Получение и настройка SSL-сертификата

>В этом разделе описаны шаги для получения SSL-сертификата от бесплатного центра
сертификации Let's Encrypt

 Находясь на сервере, установите certbot, если он ещё не установлен: 
 
 ```
 # Установка пакетного менеджера snap.
sudo apt install snapd
# Установка и обновление зависимостей для пакетного менеджера snap.
sudo snap install core; sudo snap refresh core
# Установка пакета certbot.
sudo snap install --classic certbot
# Обеспечение доступа к пакету для пользователя с правами администратора.
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Запустите certbot и получите SSL-сертификат. Для этого выполните команду:

```
sudo certbot --nginx
```

Проверьте конфигурацию Nginx, и если всё в порядке, перезагрузите её.
