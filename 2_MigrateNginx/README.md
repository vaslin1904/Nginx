# Миграция с nginx на Angie #
______________________________________________________________________________________________
## 1. Для пакетов
* Собрать информацию об установленном Angie:
  - о версии пакета
  - о местоположении конфигов
  - о подключенных модулях </br>
**angie -V** </br>
![info_angie](picture/inf%20angie.png) </br>
* Скопировать конфиги nginx в папку /etc/angie, кроме nginx.conf </br>
  В данной работе скопированы папки sites-available, sites-enabled, snipets,static-avif.conf.</br>
![tree](picture/inf%20angie.png) </br>
* Из конфига nginx перенести данные:</br>
  - Пользователь: user www-data;</br>
  - Перенос настроек proxy в директиве http</br>
  - Перенос настроек gzip</br>
  - Перенос настроек brotly, ранее был установлен и вкючен в конфиге angie</br>
  - Переносится include в http модуле. Подключение директории /etc/angie/sites-enabled/*</br>
  - Перенос настроек map
* Поиск в скопированных конфигах "nginx" и замена на angie</br>
  **grep -nr "nginx"**</br>
   - Для одного файла: **sed -i 's|nginx|angie|g' snippets/fastcgi-php.conf** </br>
  - Для всех файлов где встречается "nginx": **find . -type f -name '*' exec sed -i 's|/nginx|/angie|g' {} \;**</br>
![grep_nginx.png](picture/change%20nginx%20in%20conf.png) 
*Изменить путь в символических ссылках, замена nginx на angie</br>
*/sudo find . -type l -name "default" -exec sh -c ' </br>
  for f; do </br>
    old=$(readlink "$f") </br>
    new=$(echo "$old" | sed "s|/nginx|/angie|g") </br>
#n-блокирует разыменование, f -удаление старой ссылки, s - создание новой ссылки</br>
    ln -sfn "$new" "$f" </br>
  done </br>
' _ {} + /* </br>
![change](picture/3%20change%20link.png)
* Проверка конфигурации</br>
  **angie -t**</br>
Столкнулась с ошибкой: */ angie: [emerg] unknown "vary_header" variable</br>
                        angie: configuration file /etc/angie/angie.conf test failed/*</br> 4 Ошибка переменной в конфигах.png
 ![error](picture/4%Ошибка%переменной%в%конфигах.png)                       
  Решение:</br>
  Найти в каком файле есть такое сочетание "vary_header"</br> 
  **sudo find /etc/angie -type f -name "*.conf" -exec grep -l "vary_header" {} + **</br>
  В результате выполнения найден скопированный файл nginx static-avif.conf.</br>
 static-avif.conf - для настройки обработки запросов к изображениям в форматах AVIF и WebP.</br>
![static-avif.conf](picture/inf%20angie.png) </br>
  Ошибка связа с тем, что не был перенесен с конфига nginx в angie.conf блок настроек **map**</br>
  ![map](picture/5.%20map%20from%20nginx.png)
  при этом был перенесен файл static-avif.conf.</br>
* Сравнить файлы mem.types, добавить изменения в файл angie</br>
* Проверить права для файлов</br>
* Проверка конфигурации</br>
  **angie -t**</br>
* Перечитать конфиги angie</br>
**sudo kill -HUP $(cat /run/angie.pid)**</br>
![map](picture/6%20reload%20configs%20angie%20HUP.png)
* Если есть в системе nginx!!!</br>
- Обязательно остановить nginx перед запуском angie</br>
  **sudo systemctl nginx stop && sudo systemctl angie start**</br>
- Проверить есть ли автозапуск у nginx. !!!! Отключить</br>
**sudo systemctl disable nginx**</br>
- Включить автозапуск у Angie</br>
**sudo angie enable**</br>
![status](picture/status.png)
__________________________________________________________________________________________________________________
## Миграция с Docker
*
