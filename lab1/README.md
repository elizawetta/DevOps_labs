## Настроить nginx по заданному тз:
1. Должен работать по https c сертификатом
2. Настроить принудительное перенаправление HTTP-запросов (порт 80) на HTTPS (порт 443) для обеспечения безопасного соединения.
3. Использовать alias для создания псевдонимов путей к файлам или каталогам на сервере.
4. Настроить виртуальные хосты для обслуживания нескольких доменных имен на одном сервере.

### Подготовка
Устанавливаем `nginx`

Для создание сертификата устанавливаем `openssl` и создаем сертификат. 

```sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/selfsigned.key -out /etc/ssl/certs/selfsigned.key```
![](./files/first.png)

Создаем папки
```sudo mkdir -p /etc/nginx/sites-available /etc/nginx/sites-enabled```
Там создаем два файла `site1.com` и `site2.com`, в них прописываем перенаправление на нужный порт, а также только что созданный сертификат для https. Задаем корневые папки для каждого проекта

`site1.com`
```
server {
    listen 80;
    server_name site1.com www.site1.com;
    
    return 301 https://$host$request_uri;
}

server {
    listen 443 sl;
    server_name site1.com www.site1.com;
    
    ssl_certificate /etc/ssl/certs/selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/selfsigned.key; 
    
    root /var/www/site1;
}
```

`site2.com`
```
server {
    listen 80;
    server_name site2.com www.site2.com;
    
    return 301 https://$host$request_uri;
}

server {
    listen 443 sl;
    server_name site2.com www.site2.com;
    
    ssl_certificate /etc/ssl/certs/selfsigned.crt;
    ssl_certificate_key /etc/ssl/private/selfsigned.key; 
    
    root /var/www/site2;
}
```

Также создаем файлы `index.html` в `/var/www/site2` и `/var/www/site1`

Добавляем в `/etc/hosts` доменные имена `site1` и `site2` чтобы их можно было открыть на сервере. 