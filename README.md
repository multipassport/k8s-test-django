# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

## Как запустить сайт с помощью Kubernetes

Установите [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/)

Установите [minikube](https://minikube.sigs.k8s.io/docs/start/)

Установите [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Запустите [кластер minikube](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/)

В файле `config-map.yml` замените значения переменных окружения на ваши, где `DATABASE_URL` - адрес базы данных PostgreSQL, `ALLOWED_HOSTS` - адрес кластера minikube, полученный с помощью команды `minikube ip`

Запустите следующую команду, чтобы добавить в файл /etc/hosts переадресацию запросов с введенного вами url на IP кластера minikube, где my-site.test можно заменить на нужный url
```shell-session
$ echo "$(minikube ip) my-site.test" | sudo tee -a /etc/hosts
```

После этого запустите deployment kubectl командой
```shell-session
$ kubectl apply -f django-service.yml
```

Запустите миграции базы данных командой
```shell-session
$ kubectl apply -f django-migrate-job.yml
```

И включите cronjob kubectl командой
```shell-session
$ kubectl apply -f cronjob.yml
```

## Переменные окружения

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).
