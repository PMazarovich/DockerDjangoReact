# Taski

Taski — это приложение для планирования задач. Оно объединяет фронтенд (React) и бэкенд (Django) с использованием Nginx для обработки статических файлов.

## Особенности

- **Frontend**: Собранный React-приложение, оптимизированное для работы через Nginx.
- **Backend**: Django, включая сбор и раздачу статических файлов.
- **Gateway**: Отвечает за маршрутизацию запросов и раздачу статики.

## Конфигурация Nginx

```nginx
location / {
    alias /staticfiles/;
    index index.html;
}
```

### Пояснение

- **Маршруты API и админки**: Запросы, начинающиеся с `/api` или `/admin`, перенаправляются к бэкенду через `proxy_pass`.

  ```nginx
  location /api {
      proxy_pass http://backend:8000/api/;
  }

  location /admin {
      proxy_pass http://backend:8000/admin/;
  }
  ```

- **Статические файлы**: Nginx ищет файлы по пути, указанному в запросе пользователя. Если файл не найден, возвращается `index.html`.

  Пример:
    - Запрос: `http://localhost:8000/static/rest_framework/css/bootstrap.min.css`
    - Nginx ищет файл по пути `/staticfiles/static/rest_framework/css/bootstrap.min.css`.
    - Если файл отсутствует, возвращается `index.html`.

## Структура каталогов

```plaintext
/staticfiles/
|-- asset-manifest.json <-- cтатитка react 
|-- favicon.ico         <-- cтатитка react 
|-- index.html          <-- cтатитка react 
|-- logo192.png         <-- cтатитка react 
|-- logo512.png         <-- cтатитка react 
|-- manifest.json       <-- cтатитка react 
|-- robots.txt          <-- cтатитка react 
`-- static
    |-- admin/  <-- cтатитка admin панели drf 
    |-- css/    <-- cтатитка react 
    |-- js/     <-- cтатитка react
    |-- rest_framework/ <-- cтатитка drf
```

## Сборка проекта
Все опресции сборки происходят из докерфайлов и переносятся в соответствующий volume /static
См. docker-compose и dockerfiles
1. **Собрать фронтенд**:

   ```bash
   npm run build
   ```
   Переместить содержимое папки `build/` в `/staticfiles/`.

2. **Собрать статику для бэкенда**:

   ```bash
   python manage.py collectstatic
   ```
    - Переместить файлы в `/staticfiles/static/`.
    - Убедиться, что:
        - В `settings.py` указан:
          ```python
          STATIC_URL = 'static'
          ```
        - Таким образом будет создан префикс `/static/` для всех файлов, необходимых для Django.
        - Все файлы будут искаться в соответствующей папке в nginx контейнере.
3. **Настроить Nginx**: Убедитесь, что конфигурация Nginx указывает на правильные каталоги.

## Пример работы

- Запрос: `http://localhost:8000/api/`
    - Приходит в контейнер nginx на порт 8000
    - Перенаправляется на `http://backend:8000/api/`.
    - Бэкенд возвращает HTML-файл, который включает:

      ```html
      <script src="/static/rest_framework/js/jquery-3.5.1.min.js"></script>
      <script src="/static/rest_framework/js/ajax-form.js"></script>
      <script src="/static/rest_framework/js/csrf.js"></script>
      <script src="/static/rest_framework/js/bootstrap.min.js"></script>
      <script src="/static/rest_framework/js/prettify-min.js"></script>
      <script src="/static/rest_framework/js/default.js"></script>
      ```

- Эти файлы загружаются через Nginx из `/staticfiles/static/rest_framework/`. Если файлы отсутствуют, возвращается `index.html` для обработки SPA.

- Запрос: `http://localhost:8000/`
    - Приходит в контейнер nginx на порт 8000
    - Перенаправляется на путь '/' внутри nginx.
    - Nginx отдает index.html, который инициирует загрузку из клиентского браузера этих файлов:

      ```html
      <script defer="defer" src="/static/js/main.c77806b7.js"></script>
      <link href="/static/css/main.f03fe314.css" rel="stylesheet">
      ```

- Эти файлы загружаются через Nginx из `/staticfiles/static/`. 


## Github actions
См. в .github/workflows/main.yaml

Обратить внимание docker compose и Dockerfile для обоих сервисов. Так можно понять, что происходит в main.yaml и почему.

# Как настроить nginx, доменное имя и сертификат на удаленной машине?
## **Установка и запуск Nginx**

1. **Установите Nginx:**
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```

2. **Запустите службу Nginx:**
   ```bash
   sudo systemctl start nginx
   ```

3. **Откройте доступ для Nginx и SSH:**
   ```bash
   sudo ufw allow 'Nginx Full'
   sudo ufw allow OpenSSH
   ```

4. **Включите файервол:**
   ```bash
   sudo ufw enable
   ```

5. **Проверьте статус:**
   ```bash
   sudo ufw status
   ```

---

## **Настройка конфигурации Nginx**

1. **Откройте файл конфигурации:**
   ```bash
   sudo nano /etc/nginx/sites-enabled/default
   ```

2. **Добавьте конфигурацию:**
   ```nginx
   server {
       listen 80;
       server_name example.com www.example.com;

       root /var/www/html;
       index index.html index.htm;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```
   Подробнее см. в sample_nginx.conf

3. **Проверьте корректность конфигурации:**
   ```bash
   sudo nginx -t
   ```

4. **Перезагрузите Nginx:**
   ```bash
   sudo systemctl reload nginx
   ```

---

## **Установка SSL-сертификата с Certbot**

1. **Установите Certbot:**
   ```bash
   sudo snap install --classic certbot
   ```

2. **Настройте SSL автоматически:**
   ```bash
   sudo certbot --nginx
   ```

    - Введите ваш домен при запросе.
    - Подтвердите установку сертификата.

3. **Проверьте список сертификатов:**
   ```bash
   sudo certbot certificates
   ```

---

## **Обновление SSL-сертификатов**

Чтобы проверить обновление сертификатов:
```bash
sudo certbot renew --dry-run
```

---

## **Дополнительные команды**

- **Проверить статус Nginx:**
  ```bash
  sudo systemctl status nginx
  ```

- **Перезапустить Nginx:**
  ```bash
  sudo systemctl restart nginx
  ```