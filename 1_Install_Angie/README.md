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

2. Загрузка образа docker и создание контейнера
*docker run --rm --name angie -v /var/www:/usr/share/angie/html:ro -p **8080:80** -d docker.angie.software/angie:1.11.4-ubuntu*
![angie in docker](picture/1psdocker_ang.png) </br>
3. Просмотр занятых портов  angie (port 82) и docker (8080)
   *ss -ntpl*
![ports](picture/2ntlpdoc_ang.png) </br>
4. Проверить работу Angie на порту 82 и 80880
*curl http://localhost:82*
*curl http://localhost:8080*
![curl](picture/3doc_curl.png) </br>
5. Скопировать конфигурационные файлы из докер на хостовую машину. Присоединить полученную папку к контейнеру</br>
   *sudo docker cp angie:/etc/angie/ /home/vagrant/angie*
   *sudo docker run --name ang_vol -v /var/www:/usr/share/angie/html:ro -v /home/vagrant/angie:/etc/angie:ro --network host -d docker.angie.software/angie:1.11.4-ubuntu*
6. Узнать какая команда использовалась для создания контейнера
   
   


