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
В данном случае маппинг домашней директории /var/jenkins_home предполагает, что сначала доке проверит в текущем каталоге наличие папки jenkins_data.
Если папки нет, она будет создана.

#### Перезапуск докер контейнера:
```bash
docker-compose down
docker-compose up -d
```
Мы снова наблюдаем Jenkins по адресу http://0.0.0.0:8083/ с нашими настройками и можем залогиниться. Но если удалить папку jenkins_data, тогда при перезапуске мы снова увидим дефолтное состояние Jenkins. 

#### Проблематика:
- Мы по прежнему хотим понять Jenkins в нужном нам состоянии, но никак не завязываясь на папку jenkins_data. Опубликовать образ в докер-хаб и ссылать на него.




[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

[link]: <https://otus.ru/learning/102096/>
[Jenkins-docker-compose]: <https://adamtheautomator.com/jenkins-docker/>
[ngrok-docker-compose]: <https://github.com/shkoliar/docker-ngrok>
