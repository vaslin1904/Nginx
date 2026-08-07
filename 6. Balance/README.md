# Задание. Настроить балансировку по следующим вариантам:
## 1. Равномерная балансировка (round-robin).
## 2. Балансировка по хэшу с использованием переменных (на выбор).
## 3. Произвольная балансировка (random).

## Показать варианты конфигурации с резервным бэкендом и с отключением одного из бэкендов.
__________________________________________________________________________________________________________
## Тестовая среда.
На виртуальной машине с помощью [docker-compose.yml] (./docker-compose.yml)</br>
подняты backend vscoder/webdebugger - 4 ед., webserver - angie.</br>
Для мониторинга добавлена консоль Angie Light</br>
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
__________________________________________________________________________________________________________</br>
## Настройка Upstream
В блок http добавлен блок upstream:</br>
``` upstream backend {
  zone upstream-backend 10m;
   server debug-white:8080;
   server debug-blue:8080;
   server debug-green:8080;
   server debug-gold:8080;
}
```
В блоке server добавлено перенаправление на **backend**</br>

``` location / {
       proxy_pass http://backend;
       proxy_http_version 1.1;
       proxy_set_header Connection "";
    }
```
_____________________________________________________________________________________________________________
### Round-Robin
Данный тип балансировки по умолчанию.</br>
Каждый сервер получает примерно одинаковое количество запросов.</br>
<img width="1314" height="684" alt="image" src="https://github.com/user-attachments/assets/e5c7c32d-0a10-4885-8720-35b1efe4fc5f" /></br>
Выводим один из серверов в состояние down
```server debug-white:8080 down;
```
<img width="1179" height="565" alt="image" src="https://github.com/user-attachments/assets/23df4e54-7960-4463-9e83-913e8b9a24ab" /></br>
_____________________________________________________________________________________________________________</br>
### HASH
Распределение серверов по значению хэша от переменной</br>
Параметр **consistent** указывает на использование консистентное хэширования, при котором при отключении серверов минимальное количество </br> клиентов переезжает на другие сервера.</br>
``` upstream backend {
  zone upstream-backend 10m;
  hash $host consistent;
   server debug-white:8080;
   server debug-blue:8080;
   server debug-green:8080;
   server debug-gold:8080;
}
```
<img width="1194" height="689" alt="image" src="https://github.com/user-attachments/assets/9cea0f8d-7e9c-44e2-8853-dac2e7890cff" /></br>
```server debug-green:8080 down;
```
<img width="1186" height="610" alt="image" src="https://github.com/user-attachments/assets/9f5f3eb9-53a2-40c6-8b91-80fd59f56675" /></br>

___________________________________________________________________________________________________________________________________</br>
### Random
Произвольный выбор сервера</br>
```upstream backend {
  zone upstream-backend 10m;
  random; 
   server debug-white:8080;
   server debug-blue:8080;
   server debug-green:8080;
   server debug-gold:8080;
}
```
<img width="1186" height="669" alt="image" src="https://github.com/user-attachments/assets/22a8240f-dd3f-4864-a78f-021c8fbc7106" /></br>
_________________________________________________________________________________________________________________________________</br>
### Настройка Резервного сервера
Если upstream перестанут отвечать, то активным станет резервный сервер</br>
На резервный сервер добавляется метка</br>
```upstream backend {
  zone upstream-backend 10m;
   server debug-white:8080;
   server debug-blue:8080;
   server debug-green:8080 backup;
   server debug-gold:8080;
}
```
Добавляем переключение между upstream</br>
``` location / {
       proxy_pass http://backend;
       proxy_http_version 1.1;
       proxy_set_header Connection "";
       proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
       proxy_next_upstream_tries 1;
       proxy_next_upstream_timeout 10s;
    
    }
```
<img width="1760" height="707" alt="image" src="https://github.com/user-attachments/assets/4369b222-bb8e-4714-9d40-78f0693a966e" /></br>

<img width="1065" height="707" alt="image" src="https://github.com/user-attachments/assets/0952e2f9-4a05-49a2-9324-9293ba5092bf" /> </br>
