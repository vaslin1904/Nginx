# Nginx/Angie
## Часть1. Установка Angie из пакетов
1. Установка пакета для работы с сертифкатами </br>
      *sudo apt-get update </br>
      sudo apt-get install -y ca-certificates curl*</br>
2. Получение открытого ключ репозитория Angie для проверки подлинности пакетов</br>
     *sudo curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg \ </br>
            https://angie.software/keys/angie-signing.gpg* </br>
3. Подключение репозитория </br>
    *echo "deb https://download.angie.software/angie/$(. /etc/os-release && echo "$ID/$VERSION_ID $VERSION_CODENAME") main" \</br>
    | sudo tee /etc/apt/sources.list.d/angie.list > /dev/null* </br>
   *sudo apt-get update* </br>
4. Установка Angie и дополнительного модуля brotli </br>
   *sudo apt install angie angie-module-brotli* </br>

-------------------------------------------------------------------------------------------------------------------------------
## Brotli</br>
  ngx_brotli_filter — используется для сжатия ответов на лету.</br>
  ngx_brotli_static — используется для обработки предварительно сжатых файлов.</br>
**Подключение модуля к Angie в config**</br>
Подключение модулей в контексте main{}:</br>
  + load_module modules/ngx_http_brotli_filter_module.so;</br>
  + load_module modules/ngx_http_brotli_static_module.so;</br> \
![Правка /etc/angie/angie.conf](picture/load_brodli_in_config.png)</br>
_______________________________________________________________________________________________________________________________
Проверка работы Angie \
+ ps -afx
![ps](picture/ps_afx_ang.png)</br>
+ sudo systemctl status angie.service \
+ ![systemctl status](picture/status_service_ang.png) </br>
_______________________________________________________________________________________________________________________________
## Часть2. Запуск Angie через Docker
1. Установка  docker
   *apt install docker.io*

2.Загрузка образа docker и создание контейнера
docker run --rm --name angie -v /var/www:/usr/share/angie/html:ro -p 8080:80 -d docker.angie.software/angie:1.11.4-ubuntu







