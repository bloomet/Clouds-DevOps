# Лабораторная №1

## Задание:
Настроить nginx по заданному тз:
1) Должен работать по https c сертификатом
2) Настроить принудительное перенаправление HTTP-запросов (порт 80) на HTTPS (порт 443) для обеспечения безопасного соединения.
3) Использовать alias для создания псевдонимов путей к файлам или каталогам на сервере.
4) Настроить виртуальные хосты для обслуживания нескольких доменных имен на одном сервере.
5) Что угодно еще под требования проекта

## Ход выполнения

Для начала установим веб-сервер Nginx на виртуальной машине с ОС Linux (Ubuntu). Сделать это можно с помощью следующих команд:

```bash
sudo apt update
sudo apt install nginx
```

Запустим его командой ```sudo systemctl start nginx``` и проверим, что все корректно отображается. Как мы видим, все гуд!

<img width="1143" height="300" alt="image" src="https://github.com/user-attachments/assets/f470e3d6-8c5e-4917-b373-01295447c0a2" />

Затем создадим каталоги для проектов и настроим права доступа с помощью ```sudo chown``` и других команд, изображенных на скриншоте:

<img width="812" height="244" alt="image" src="https://github.com/user-attachments/assets/99437819-9bc6-49fc-8675-546829baf1f5" />


Следующим шагом будет установка ssl и генерация самоподписанного сертификата. SSL устанавливается командой ```sudo apt install openssl```, 
а сертификат генерируется с помощью ```sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt```.
Процесс его настройки изображен на скриншоте:

<img width="877" height="226" alt="image" src="https://github.com/user-attachments/assets/eeb937c2-f0c1-4b67-9112-c8776f22cb5d" />

Когда мы приготовили все инструменты, настало время приступать к самой лабораторной. Будем настраивать виртуальные хосты,
для этого пропишем ```sudo nano /etc/nginx/sites-available/proj1```, у нас откроется конфигурация виртуального хоста первого проекта,
в ней мы должны написать следующее:

```nginx
server {
    listen 80;
    server_name proj1.example.com;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name proj1.example.com;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    location / {
        root /var/www/proj1;
        index index.html;
    }
}
```

В ней мы указали доменное имя (server_name), перенаправление с http на https, путь к ssl сертификату, который мы подписали
заранее, указали, где искать файлы проекта (root /var/www/proj1) и установили index.html в качестве главной страницы сайта.
Со вторым проектом все аналогично, поэтому просто оставим его конфиг здесь:

```nginx
server {
    listen 80;
    server_name proj2.example.com;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name proj2.example.com;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    location / {
        root /var/www/proj2;
        index index.html;
    }
}
```

После внесения этих изменений файлы нужно активировать, создав символические ссылки в директории /etc/nginx/sites-enabled/
с помощью следующих команд:

```bash
sudo ln -s /etc/nginx/sites-available/proj1 /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/proj2 /etc/nginx/sites-enabled/
```

Далее перезагрузим nginx командой ```sudo systemctl restart nginx``` для того, чтобы все изменения вступили в силу. Теперь
нам надо узнать IP-адрес нашей машины, на которой установлен nginx. Делается это с помощью команды ```hostname -I```. Этот
IP-адрес будет использоваться для связи между нашим компьютером и виртуальными хостами Nginx.

Следующим шагом нам надо будет отредактировать /etc/hosts, который используется для сопоставления доменных имен и IP-адресов
на локальном уровне, минуя DNS-серверы. Открываем его с помощью ```sudo nano /etc/hosts``` и добавляем в него следующие строки:

```
your_server_ip  proj1.example.com
your_server_ip  proj2.example.com
```

На месте ```your_server_ip``` должен быть адрес нашей виртуальной машины, а proj1.example.com и proj2.example.com будут 
доменными именами наших проектов. Ниже прикреплен скриншот того, как выглядит наш hosts:

<img width="679" height="464" alt="image" src="https://github.com/user-attachments/assets/316c9b7e-b3dd-47e7-9b43-91cf2e79c5b6" />

Далее переходим в браузере по адресам [Project 1](http://proj1.example.com) и [Project 2](http://proj2.example.com), и, как видим, все работает +
реализовано автоматическое перенаправление на https.

<img width="489" height="497" alt="image" src="https://github.com/user-attachments/assets/33faaab4-1ad9-404a-af38-2dd5ceb7d73d" />

<img width="494" height="488" alt="image" src="https://github.com/user-attachments/assets/6f91af66-561a-4695-bcff-f89c5c844b6a" />

Thanks for watching! Это было трудно...
