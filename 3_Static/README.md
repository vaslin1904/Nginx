# Задание
* Запустить приложение на сервере.
* Для каждой директории создать location со своими настройками.
* Отдельно создать location с регулярным выражением для отдачи картинок jpg, jpeg, png, gif.
* Задействовать переменные, определённые с map для работы с location.
* Настроить два вида перенаправлений (301/302 и внутренние).
___________________________________________________________________________________________________
Запуск приложения.
- Создается контейнер, в которы мантируются папки с содержанием сайта и папка angie с содержанием конфигов.
  **docker run --name=angie --volume /home/vasy/angie:/etc/angie:ro --volume /var/www:/usr/share/angie/html:ro --network=host --runtime=runc --detach=true docker.angie.software/angie:1.11.4-ubuntu**
- В каталог хостовой машины размещаются папки сайта и файл index.html
- В конфиге **angie/http.d/default.conf** создаются location согласно заданию
  * location для всех запросов, если путь не найден то выдается ошибка
    *location / { </br>
      try_files $uri $uri/ =404;  </br>   
    }* </br>
  _______________________________________________________
  * location Префиксный. Для запроса к /assets сохраняются кэш 30 дней  </br>
    *location /assets/ { </br>
       expires 30d;  </br>
   }* </br>
   ______________________________________________________
 * location c регулярным выражением поиска картинок</br>
    *location ~* \.(?:jpeg|jpg|png|svg)$ { </br>
        expires 30d;	
        add_header X-Info $map_var; </br>       
   }* </br>
___________________________________________________________
 * Префиксный location без поиска в регулярках. 
    *location ^~ /assets/css/ { </br> 
       add_header X-Info $map_var; </br> 
   }* </br>
   _________________________________________________________
   *location Точное совпадение
     *location = /error/index.html { </br> 
        internal;</br> 
    }*</br> 
- Для работы с переменной map необходимо присвоить ей значение в директиве map основного конфига **angie/angie.conf**
  *map $uri $map_var { </br>
    default "standart"; </br>
    \~*jpg$ "picture"; </br>
    \~*css$ "style"; </br>
    }* </br>
  Данная переменная **map_var** по-умолчанию имеет значение **standart**,</br>
  если происходит обращение к файлу с расширением jpg, то она принимает значение **picture**</br>
  если происходит обращение к файлу с расшерением css, то она принимает значение **style**</br>
    Сама переменная используется в файле **angie/http.d/default.conf** в location.</br>
  Если запрос к происходит запрос к файлу css или jpg, то к запросу добавляется заголовок со значением переменной  **map_var**</br>
     *location ~* \.(?:jpeg|jpg|png|svg)$ { </br>
        expires 30d;	</br>
        add_header X-Info **$map_var**; </br>       
   }* </br>
  и </br>
    *location ^~ /assets/css/ { </br> 
       add_header X-Info **$map_var**; </br> 
     }* </br>
![map_css](https://github.com/vaslin1904/Nginx/blob/main/3_Static/img/map_css.png) </br> 
![map_jpg](https://github.com/vaslin1904/Nginx/blob/main/3_Static/img/map_jpg.png) </br> 
- Настроен вида перенаправлений 301/302
  *return 301 http://mysite.ru$request_uri;*
![return301/302](https://github.com/vaslin1904/Nginx/blob/main/3_Static/img/return302.png)
![img_return301/302](https://github.com/vaslin1904/Nginx/blob/main/3_Static/img/return302_2.png)
_______________________________________________________________________________________________________________________
Сайт отображается со всеми css, ссылками
![site](https://github.com/vaslin1904/Nginx/blob/main/3_Static/img/site.png)
  
