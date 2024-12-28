# Taski
Приложение для планирования своих задач.
Разъяснение по статике.
Ngnix может самостоятельно раздавать статические файлы.
location / {
alias /staticfiles/;
index index.html;
}
Это значит, что для дюбого запроса, который не /api и не /admin, попытаться возвратить файл по пути GET запроса пользователя
Пример:
http://localhost:8000/api/ - этот запрос пришел в nginx контейнер.
/api распределился на /backend адрес -> proxy_pass http://backend:8000/api/;
Этот get запрос вернул некий html файл, который включает в себя, в частности,
<script src="/static/rest_framework/js/jquery-3.5.1.min.js"></script>
<script src="/static/rest_framework/js/ajax-form.js"></script>
<script src="/static/rest_framework/js/csrf.js"></script>
<script src="/static/rest_framework/js/bootstrap.min.js"></script>
<script src="/static/rest_framework/js/prettify-min.js"></script>
<script src="/static/rest_framework/js/default.js"></script>

Это заставит браузер запросить все эти файлы по таким адресам: 
http://localhost:8000/static/rest_framework/css/bootstrap.min.css
http://localhost:8000/static/rest_framework/css/bootstrap-tweaks.css
http://localhost:8000/static/rest_framework/js/default.js
И тд.
Они тут же начнут свое исполнение и могут дососать с сервера еще какие-то файлы.
Эти запросы, к примеру http://localhost:8000/static/rest_framework/css/bootstrap.min.css никак не отражены в ngnix.conf
По умолчанию, они перенаправляются в блок
location / {
    alias /staticfiles/;  <-- искать в первую очередь в этой папке
    index index.html;   <-- если в папке ничего не нашли, из этой же папки возвращаем index.html
}
Т.е. ngnix попробует в папке staticfiles найти этот путь /static/rest_framework/css/bootstrap.min.css и отдать его.
Если его нет, он отдаст файл index.html, который уже является частью SPA.
После отдачи index.html, начнется засасывание всех файлов SPA
Структура volume ngnix.conf:
-- Это всё от SPA React
|-- asset-manifest.json
|-- favicon.ico
|-- index.html
|-- logo192.png
|-- logo512.png
|-- manifest.json
|-- robots.txt
-- Это - статика Django
`-- static
    |-- admin
    |-- css
    |-- js
    `-- rest_framework
----------

Предварительно, естественно, нужно собрать frontend (npm run build) и положить в папку /staticfiles/ nginx
И собрать бэкэнд (python manage.py collectstatic) и положить в папку staticfiles/static/ nginx