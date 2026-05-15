# Nginx/Angie
## Установка Angie из пакетов
1. Установка пакета для работы с сертифкатами
      *sudo apt-get update
      sudo apt-get install -y ca-certificates curl*
2. Получение открытого ключ репозитория Angie для проверки подлинности пакетов
     *sudo curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg \
            https://angie.software/keys/angie-signing.gpg*
3. Подключение репозитория
    *echo "deb https://download.angie.software/angie/$(. /etc/os-release && echo "$ID/$VERSION_ID $VERSION_CODENAME") main" \
    | sudo tee /etc/apt/sources.list.d/angie.list > /dev/null*
   *sudo apt-get update*
4. Установка Angie и дополнительного модуля brotli
   *sudo apt install angie angie-module-brotli*

-------------------------------------------------------------------------------------------------------------------------------
## Brotli
ngx_brotli_filter — используется для сжатия ответов на лету.
ngx_brotli_static — используется для обработки предварительно сжатых файлов.
**Подключение модуля к Angie в config**
Подключение модулей в контексте main{}:
 load_module modules/ngx_http_brotli_filter_module.so;
 load_module modules/ngx_http_brotli_static_module.so;
[Правка /etc/angie/angie.conf][picture/load_brodli_in_config.png]










 -

