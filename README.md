# Как развернуть Yii2 проект на Heroku

На самом деле не стоит использовать Yii2, когда есть нормальный фреймворк	[Symfony](https://symfony.com/).

Что необходимо иметь в арсенале:
- PHP
- git
- Браузер
- Репозиторий с Yii2 в [GitLab](https://gitlab.com/).

Первым делом регистрируемся на [Heroku](https://www.heroku.com/).

После успешной регистрации создаём новое приложение на Heroku:

![Screenshot](/screenshots/Screenshot_1.png)

Вводим название и нажимаем кнопку "Create app":

![Screenshot](/screenshots/Screenshot_2.png)

Затем нажимаем на вкладку "Overview":

![Screenshot](/screenshots/Screenshot_3.png)

Вводим в поле поиска `postgres` и выбираем "Heroku Postgres":

![Screenshot](/screenshots/Screenshot_4.png)

Нажимаем "Provision" и тем самым создаём сервер бд:

![Screenshot](/screenshots/Screenshot_5.png)

Теперь редактируем `composer.json`:
- Добавляем `"ext-gd": "*"` в require
- Переносим `"yiisoft/yii2-debug": "~2.1.0"` и `"yiisoft/yii2-gii": "~2.1.0"` из require-dev в require

![Screenshot](/screenshots/Screenshot_6.png)

Теперь заходим в GitLab и нажимакм "Set up CI/CD":

![Screenshot](/screenshots/Screenshot_7.png)

Перед нами открывается окно настройки CI/CD:

![Screenshot](/screenshots/Screenshot_8.png)

Вставляем:
```
image: php:7.2

before_script:
  - apt-get update -qq
  - apt-get install -y -qq git

deploy to heroku:
  stage: deploy
  script:
  - apt-get install -y ruby
  - gem install dpl
  - dpl --provider=heroku --app=ТУТ_НАЗВАНИЕ_ПРОЕКТА --api-key=ТУТ_КЛЮЧ
```

# Где взять api-key?
Открываем Heroku, нажимаем на аватар, нажимаем "Account settings":

![Screenshot](/screenshots/Screenshot_9.png)

Скроллим до API Key, нажимаем Reveal и копируем ключ и вставляем в `-api-key`

![Screenshot](/screenshots/Screenshot_10.png)

Теперь нажимаем "Commit changes" из пункта выше.

Нажимаем на CI/CD, нажимаем выбираем pipeline:

![Screenshot](/screenshots/Screenshot_11.png)

Смотрим логи деплоя и наслаждаемся деплоем не самого лучшего фреймворка:

![Screenshot](/screenshots/Screenshot_12.png)

В конце должно быть сообщение "Job succeded" и ссылка на Heroku:

![Screenshot](/screenshots/Screenshot_13.png)

Переходим по ссылке и видим, что получилось сделать автодеплой с мастера:

![Screenshot](/screenshots/Screenshot_14.png)

# Настраиваем коннект с БД

Открываем вкладку "Overview" на Heroku и нажимаем на "Heroku Postgres":

![Screenshot](/screenshots/Screenshot_15.png)

Открываем вкладку "Settings" нажимаем на "View credentials":

![Screenshot](/screenshots/Screenshot_16.png)

Редактируем файл `config/bd.php` и вставляем данные для бд:

![Screenshot](/screenshots/Screenshot_17.png)

Пушим измненения, ждём пока проект задеплоится.

# Настраиваем nginx
Создаём файл `Procfile` и вставляем туда:
`web: vendor/bin/heroku-php-nginx -C nginx.conf web`
Создаем файл `nginx.conf` и вставляем туда:
```
location / {
    index index.php;
    try_files $uri $uri/ /index.php?$args;
}
```
Пушим измненения, ждём пока проект задеплоится.

# Optional
Можно добавить, чтобы миграции накатывались автоматом и перечислить команды для работы с фикстурами, но почему-то возникает ошибка на Heroku.

Для этого создаём файл ```post-install-cmd.sh``` и вставляем:
```
#!/bin/sh
yii migrate --interactive=0
ТУТ_КОМАНДЫ_ДЛЯ_РАБОТЫ_С_ФИКСТУРАМИ
```

В composer.json добавляем:
```
"scripts": {
    "post-install-cmd": [
        "sh post-install-cmd.sh"
    ]
}
```

Пушим изменения, смотрим на результаты билда.

# Запускаем миграции самостоятельно
- Проверяем, что в `config/db.php` лежат данные от БД Heroku
- Запускаем миграции: `yii migrate --interactive=0`

# Пруфы того, что есть коннект с БД:

Была создана миграция для постов:

![Screenshot](/screenshots/Screenshot_18.png)

![Screenshot](/screenshots/Screenshot_19.png)

Создан тестовый пост. Результаты вывода данных из бд:

![Screenshot](/screenshots/Screenshot_20.png)
