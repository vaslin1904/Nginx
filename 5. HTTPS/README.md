# Задание. Настроить HTTPS
_______________________________________________________________________________________
## Шаги для выполнения
1. Получить сертификат Let's Encrypt или создать самоподписной.
2. Настроить основные параметры HTTPS в Angie.
3. Оптимизировать восстановление сессий.
4. Включить автоматическую переадресацию с HTTP на HTTPS.
5. Настроить заголовок HSTS.
6. Включить протоколы HTTP/2 и HTTP/3.
7. Провести тестирование корректности конфигурации с помощью внешнего сервиса.
________________________________________________________________________________________
## Получение сертификата Let's Encrypt
Для получения сертификата был зарегистрирован бесплатный поддомен, DNS запись которого была размещена на открытом ресурсе.
 *https://freedns.afraid.org/*
Проверяем свой домен:
**dig +short mystend.strangled.net @8.8.8.8**
<img width="1024" height="60" alt="image" src="https://github.com/user-attachments/assets/4570c72d-7ebd-400a-b465-f6d7cf2e7fdc" />
Вносим новое имя сайта в конфиг [default.conf](./default.conf)
``` server {
      listen 80 default_server reuseport;
      server_name mystend.strangled.net;
```
1. Определяем в блоге http ресурс для получения сертификата rsa, ecdsa
   **acme_client rsa https://acme-v02.api.letsencrypt.org/directory key_type=rsa;
     acme_client ecdsa https://acme-v02.api.letsencrypt.org/directory;**

2. Для автоматического обновления определяем location
```location /.well-known/acme-challenge/ {
          root /var/www/acme;
      }
```
3. Настраиваем получение сертификатов
``` #Настройка получения сертификатов
   
      acme	rsa;
      acme	ecdsa;

      ssl_certificate	        $acme_cert_rsa;
      ssl_certificate_key 	$acme_cert_key_rsa;
      ssl_certificate		$acme_cert_ecdsa;
      ssl_certificate_key	$acme_cert_key_ecdsa;
```
4. Проверяем полученный сертификат
   **sudo docker exec -it webserver openssl s_client -connect mystend.strangled.net:443**
<img width="971" height="511" alt="image" src="https://github.com/user-attachments/assets/7829a468-54de-4abb-b088-b2faf9465089" />
_________________________________________________________________________________________________________________________________
## Настройка основных параметров HTTPS
Включение HTTPS по порту 443
``` listen 443 ssl reuseport;
    listen 443 quic reuseport;
```
``` #По умолчанию:
      ssl_protocols       TLSv1.2 TLSv1.3;
      ssl_ciphers         HIGH:!aNULL:!MD5;      
      ssl_session_tickets on;
```
_________________________________________________________________________________________________________________________________
## Оптимизация восстановления сессий [angie.conf](./angie.conf).
``` #Оптимизация SSL cоединений - кэширование. 
# В 1 мегабайт кэша помещается около 4000 сессий.
    ssl_session_cache   shared:SSL:4m;
    ssl_session_timeout 5m;
```
_______________________________________________________________________________________________________________________________
## Включение автоматической переадресации с HTTP на HTTPS.
``` location / {
         return 301 https://$host$request_uri;
      }
```
Проверка перенаправления:
**curl -I http://mystend.strangled.net**
<img width="630" height="227" alt="image" src="https://github.com/user-attachments/assets/093cd5ba-9535-44c2-bb2f-4fe2fa45cca9" />
Работа по https </br>
<img width="1635" height="944" alt="image" src="https://github.com/user-attachments/assets/039e5a96-80f8-4aef-b412-e8a68ba5aeae" />

______________________________________________________________________________________________________________________________
## Настройка заголовок HSTS


____________________________________________________________________________________________________________________________
## Включение протоколов HTTP/2 и HTTP/3.
``` #Подключение HTTP/2, HTTP/3
      http2 on;
      http3 on;
```
``` #Оптимизация HTTP/2
      http2_max_concurrent_streams 128;
      http2_chunk_size 8k;
#Настройка HTTP/3
      add_header Alt-Svc 'h3=":443"; ma=86400';
      http3_max_concurrent_streams 128;
```
______________________________________________________________________________________________________________________________
## Тест корректности конфигурации с помощью внешнего сервиса 
<img width="1345" height="978" alt="image" src="https://github.com/user-attachments/assets/4beff492-4140-4985-929e-e7e11a3619fa" />
