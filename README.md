# Dockerized ims.ostis

Это инструкция по запску OSTIS в [Docker](https://www.docker.com/ "Docker") контейнере. Если по каким-то причинам система не запускается локально, можно попробовать запустить её в контейнере, для этого нужно просто установить Docker и скопипастить команды ниже. Таким же образом можно запустить суперинтеллектуальную систему с винды. Это схоже с запуском на виртуалке, но весит меньше и в общем случае намного быстрее чем ставить полноценную виртуалку. Постараюсь всё расписать подробно, но если всё плохо, то в конце будет краткий гуид. Итак:

##Установка и запуск

**Шаг 0**

Нужно установить Docker:

* [Для Linux](https://www.digitalocean.com/community/tutorials/docker-ubuntu-16-04-ru "Установка Docker на Ubuntu")

* [Для Windows 10](https://hub.docker.com/editions/community/docker-ce-desktop-windows "Установка Docker на Windows 10") или [Для более старых Windows](https://docs.docker.com/toolbox/overview/ "Установка Docker на старых Windows")

Если с установкой по ссылкам что-то неясно, можно просто загуглить, там всё просто.  
Далее просто выполняем следующие команды (для шиндовс это лучше делать в bash эмуляторе типо [GitBash](https://gitforwindows.org/ "GitBash")):

**Шаг 1**

Сначала нужно скинуть всё что должно быть в базе в одну любую папку, затем зайти в эту папку в терминале, эти файлы будут скопированы в контейнер (можно рассматривать эту папку как kb, собсна туда оно всё и попадёт):
```bash
cd ostis/kb # или cd абсолютно/любая/папка/в/которой/лежат/файлы/базы
```

**Шаг 2**

Собираем образ из Dockerfile'a в этом репозитории. Клонировать его себе не обязательно.  
Тут есть два варианта: первый стянет мой образ из DockerHub'а и обновит все сабмодули, второй обновлять не будет. В первом случае контейнер будет использовать последнюю версию системы, но бывало такое что на репозитории заливались изменения которые что-то ломают. Для этого случая тут и есть второй вариант: он почти 100% запустит всё как нужно, но есть вероятность, что когда-нибудь на эти сабмодули зальют что-нибудь важное и получится так, что этот вариант будет разворачивать неактуальную версию системы.  
После обновления (или его отстутствия) оба варианта скопируют всё что есть в текущей директории в папку kb в контейнере, для этого и был нужен предыдущий шаг.  
Штош, первый вариант с обновлениями:
```bash
curl https://raw.githubusercontent.com/ARtoriouSs/ostis-docker/master/Dockerfile | docker build --pull --tag ostis --file - .
```

Или второй, без обновлений, если первый не работает:
```bash
curl https://raw.githubusercontent.com/ARtoriouSs/ostis-docker/master/Dockerfile.noupdate | docker build --pull --tag ostis --file - .
```

Тут стоит отметить, что ```.``` в конце этих команд можно заменить на путь к папке с файлами базы, тогда первый шаг не обязателен.

**Шаг 3**

Теперь у нас есть созданный локально образ, который называется ostis, это можно проверить выполнив ```docker images```, должно быть два образа artorious/ostis с пустым остисом из DockerHub'а и ostis собранный из докерфайла, который теперь нужно запустить, заэкспоузив порт 8000:
```bash
docker run -p 8000:8000/tcp ostis bash -c "./run.sh"
```

Эта команда создаст контейнер из созданного образа ostis, перенаправит порт 8000 контейнера на локальную машину и запустит внутри этого контейнера скрипт run.sh, в котором просто запускается redis-server, restart_sctp.sh и run_scweb.sh. Весь вывод этих команд будет виден в консоли, то есть если есть какие-то ошибки в при сборке базы, их можно как обычно найти там. После этого остис слушает localhost:8000, можно зайти туда в браузере и потыкать, все скопированные фалы должны быть доступны в поиске.

**Шаг 4 или как всё это вырубить шоб оно не съело всю память после нескольких перезапусков**
```bash
docker rm -f $(docker ps -aq) && docker rmi ostis
```

Эта команда удалит все контейнеры и собранный образ. Если очень хочется проверить, то ```docker ps -a```, покажет все контейнеры, они должны отсутствовать, а ```docker images``` покажет все образы, там должен остаться только базовый образ artorious/ostis. Если это всё слишком страшно (или бесполезно) и больше использоваться не будет, то можно выполнить ещё и ```docker system prune```, оно удалит вообще все образы и контейнеры, чтобы они не занимали память. Стоит помнить что каждый созданный контейнер и образ занмают память, и если не выполнять шаг 4, то будет бяка :(

Теперь можно внести изменения в свои файлы и выполнить всё сначала.

**Если всё выглядит страшно и непонятно, то вот то же самое но без пояснений:**

Собираем:
```bash
curl https://raw.githubusercontent.com/ARtoriouSs/ostis-docker/master/Dockerfile | docker build --pull --tag ostis --file - .
```
Запускаем:
```bash
docker run -p 8000:8000/tcp ostis bash -c "./run.sh"
```
Смотрим чё как,  
Вырубаем:
```bash
docker rm -f $(docker ps -aq) && docker rmi ostis
```
Вносим изменения в базу, снова собираем, запускаем...

##Автор идеи [Маша](https://github.com/idealasgas "GitHub Маши") :)
