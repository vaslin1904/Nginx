# Задание. Настроить балансировку по следующим вариантам:
## 1. Равномерная балансировка (round-robin).
## 2. Балансировка по хэшу с использованием переменных (на выбор).
## 3. Произвольная балансировка (random).

## Показать варианты конфигурации с резервным бэкендом и с отключением одного из бэкендов.
__________________________________________________________________________________________________________
## Тестовая среда.
На виртуальной машине с помощью [docker-compose.yml] (./docker-compose.yml)
подняты backend vscoder/webdebugger - 4 ед., webserver - angie.
Для мониторинга добавлена консоль Angie Light
в конфиге [default.conf](./default.conf)
``` location /console/ {
       allow 127.0.0.1;
       allow 172.16.0.0/12;
       allow 178.178.215.186;
       allow 10.0.0.0/8;
       deny all;

       auto_redirect on;
      alias /usr/share/angie-console-light/html/;
      add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";    
     }
    location /console/api/ {
        api /status/;
    }
```
__________________________________________________________________________________________________________
## Настройка Upstream
В блок http добавлен блок upstream:
``` upstream backend {
  zone upstream-backend 10m;
   server debug-white:8080;
   server debug-blue:8080;
   server debug-green:8080;
   server debug-gold:8080;
}
```
В блоке server добавлено перенаправление на **backend**

``` location / {
       proxy_pass http://backend;
       proxy_http_version 1.1;
       proxy_set_header Connection "";
    }
```
_____________________________________________________________________________________________________________
### Round-Robin
Данный тип балансировки по умолчанию.
Каждый сервер получает примерно одинаковое количество запросов.
<img width="1314" height="684" alt="image" src="https://github.com/user-attachments/assets/e5c7c32d-0a10-4885-8720-35b1efe4fc5f" />
Выводим один из серверов в состояние down
```server debug-white:8080 down;
```
<img width="1179" height="565" alt="image" src="https://github.com/user-attachments/assets/23df4e54-7960-4463-9e83-913e8b9a24ab" />
_____________________________________________________________________________________________________________
### HASH
Распределение серверов по значению хэша от переменной
Параметр **consistent** указывает на использование консистентное хэширования, при котором при отключении серверов минимальное количество клиентов переезжает на другие сервера.
``` upstream backend {
  zone upstream-backend 10m;
  hash $host consistent;
   server debug-white:8080;
   server debug-blue:8080;
   server debug-green:8080;
   server debug-gold:8080;
}
```
