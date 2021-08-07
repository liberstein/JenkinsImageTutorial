#### [Lesson from Java QA Engineer (by OTUS)][link]:
#### Проверено на окружении:
- Ubuntu 18.04.1
- Docker version 20.10.7, build f0df350
- docker-compose version 1.27.4, build 40524192  (на младших версиях синтаксис может не отработать)

#### Описание задачи:
- Требуется стартануть образ Jenkins используя docker-compose со всеми нужными настройками и джобами

#### Пример стандартного запуска:
```bash
docker-compose up -d
```
###### Начальный вид файла docker-compose.yaml:
```yaml
version: '3'
services:
  jenkins:
    image: jenkins/jenkins:2.289.3-lts-jdk11
    container_name: jenkins
    user: root
    ports:
      - "8083:8080"
      - "50003:50000"
    volumes:
      - "$PWD/jenkins_data:/var/jenkins_home"
      - "/var/run/docker.sock:/var/run/docker.sock"
```
Таким образом мы поднимаем Jenkins по адресу http://0.0.0.0:8083/. Активируем, устанавливаем плагины, меняем настройки.
В данном случае маппинг домашней директории /var/jenkins_home предполагает, что сначала докер проверит в текущем каталоге наличие папки jenkins_data.
Если папки нет, она будет создана.

#### Перезапуск докер контейнера:
```bash
docker-compose down
docker-compose up -d
```
Мы снова наблюдаем Jenkins по адресу http://0.0.0.0:8083/ с нашими настройками и можем залогиниться. Но если удалить папку jenkins_data, тогда при перезапуске мы снова увидим дефолтное состояние Jenkins. Нам же хочется поднять Jenkins сразу в нужном состоянии, но никак не завязываясь на папку jenkins_data.

#### Создадим образ jenkins-data:
В текущем каталоге у нас лежит докерфайл
```dockerfile
FROM ubuntu:18.04
RUN mkdir -p /var/jenkins_home
COPY jenkins_data /var/jenkins_home
VOLUME /var/jenkins_home
ENTRYPOINT /usr/bin/tail -f /dev/null
```
###### сбилдим образ с нужной нам папкой jenkins_data
```bash
docker build -t liberstein1/jenkinsdata06082021:1.2 .
```
###### Проверим, что образ появился:
```bash
docker images
```

###### Примерный вывод:
```bash
REPOSITORY                        TAG                 IMAGE ID       CREATED         SIZE
liberstein1/jenkinsdata06082021   1.2                 ac86eb2767da   1 hours ago    494MB
```

#### Приведем docker-compose.yaml к виду:
```yaml
version: '3'
services:
  jenkins:
    image: jenkins/jenkins:2.289.3-lts-jdk11
    container_name: jenkins
    network_mode: bridge
    user: root
    ports:
      - "8083:8080"
      - "50003:50000"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    volumes_from:
      - jenkins-data
  jenkins-data:
    image: liberstein1/jenkinsdata06082021:1.2
    container_name: jenkins-data
    network_mode: bridge
```
###### Теперь вместо того, чтобы мапить папку, мы используем volume другого контейнера.

###### Выполним снова перезапуск,:
```bash
docker-compose down
docker-compose up -d
```
###### проверим наличие активных контейнеров любой из команд на выбор:
```bash
docker ps
docker ps --format "{{json .}}"
docker ps --format "{{.ID}}\t{{.Size}}\t{{.Image}}"
```

###### Примерный вывод:
```bash
3f6181cceee6	3.35MB (virtual 684MB)	jenkins/jenkins:2.289.3-lts-jdk11
abd573947992	0B (virtual 494MB)	liberstein1/jenkinsdata06082021:1.2
```

Можно временно переименовать папку jenkins_data, которая была ранее создана при старте Jenkins в текущей директории и снова перезапуститься. Так мы убедимся, что не привязываемся к локальным настройкам.

#### Публикуем наш контейнер в docker-hub:
Сначала надо зарегистрироваться на странице [docker-hub][docker-hub]
```bash
docker login
docker push <your image name>
```

#### Критика данного подхода:
1) У нас всегда будет два контейнера вместо одного. Один - чистая версия Jenkins, второй - наш контейнер с данными.
2) Если мы загрузимся с кастомного образа из докерхаба, то мы не сможем его редактировать. Сначала потребуется поменять docker-compose файл на изначальный вид, внести необходимые изменения самом Jenkins (и соответственно в папке jenkins_data), сбилдить образ и снова опубликовать.
3) Зато можно особо не вдаваться в синтаксис докерфайлов и быстро получить результат.


[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

[link]: <https://otus.ru/learning/102096/>
[Jenkins-docker-compose]: <https://adamtheautomator.com/jenkins-docker/>
[docker-hub]: <https://hub.docker.com/>
