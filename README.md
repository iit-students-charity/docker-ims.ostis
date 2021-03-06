# Deprecation

Докеризацией OSTIS занялись официально и более серьёзно, так что скорее всего вы ищете [этот репозиторий](https://github.com/ostis-apps/dockerized-ostis). 

# Dockerized ims.ostis

Это инструкция по запску локального сервера ims.ostis в Docker контейнере.

Такой способ позволяет одной командой запускать сервер на любой операционной системе и не требует локальной установки проекта, при этом гораздо проще манипулировать несколькими базами или запускать их параллельно. Шанс появления проблем с запуском сервера сводится к минимуму, так как он происходит в заранее подготовленном изолированном контейнере.

## Установка и запуск

### Шаг 0

Установить Docker:

* [Для Linux](https://www.digitalocean.com/community/tutorials/docker-ubuntu-16-04-ru "Установка Docker на Ubuntu")
* [Для macOS](https://docs.docker.com/docker-for-mac/install/ "Установка Docker на MacOS")
* [Для Windows 10](https://hub.docker.com/editions/community/docker-ce-desktop-windows "Установка Docker на Windows 10") или [для Windows < 10](https://docs.docker.com/toolbox/overview/ "Установка Docker на Windows < 10")

При установке на Windows может потребоваться включить виртуализацию в биосе. Так же для Windows лучше использовать bash эмулятор, например [GitBash](https://gitforwindows.org/ "GitBash"). В CMD или PowerShell тоже можно. Но не нужно :)

### Шаг 1

Склонировать этот репозиторий:

```bash
git clone https://github.com/ARtoriouSs/docker-ims.ostis.git
cd docker-ims.ostis
```

### Шаг 2

Есть два варианта: запускать просто **базу / агенты на SCP** или **запускать агенты на pseudo-SCP**:

#### Для запуска базы или SCP агентов

Запустить run.sh:

```bash
./run.sh /путь/к/папке/с/базой [--noupdate] [--port=$port]
```

/путь/к/папке/с/базой — это относительный или абсолютный путь к папке, где лежат все файлы, которые должны быть в базе. Скорее всего, это будет что-то вроде ~/ostis/kb. --noupdate это _необязательный_ аргумент, который предотвратит обновление репозиториев при сборке контейнера. Его стоит использовать, если в репозитории системы попали нерабочие изменения. Состояние репозиториев образа можно найти [здесь](https://github.com/ARtoriouSs/docker-ims.ostis/blob/master/versions.md "Состояние репозиториев"). --port= это _необязательный_ аргумент, который принимает значение порта, который будет слушать сервер, например, если выполнить ```./run.sh ostis/kb --port=7000```, то сервер будет доступен на [localhost:7000](http://localhost:7000). По стандарту используется порт 8000. Можно также посмотреть `./run.sh --help`.

Чтобы выключить сервер нужно нажать ctrl + C или ctrl + Z.

После этого можно внести изменения в базу и повторить шаг 2.

#### Для запуска pseudo-SCP агентов

Запустить run_pseudo_scp.sh:

```bash
./run_pseudo_scp.sh /путь/к/папке/с/кодом [--noupdate] [--executable=$executable] [--tests-dir=$tests_dir]
```

/путь/к/папке/с/кодом — это относительный или абсолютный путь к папке, где лежит код pseudo-SCP программы, в ней должен быть файл CMakeLists.txt, а так же директория graph с тестами. Если директория с тестами называется иначе, нужно задать её название в параметре --tests-dir, но она в любом случае должна быть внутри директории с кодом (докер не позволяет копировать что-либо вне контекста сборки). Параметр --executable позволяет задать название исполняемого файла, по стандарту это wave, можно изменить в CMakeLists.txt. --noupdate, как описано выше предотвращает обновление репозиториев при сборке. Если делать всё по [методичке](https://github.com/ShunkevichDV/wave_find_path_sc_memory/blob/master/%D0%9F%D0%9F%D0%B2%D0%98%D0%A1-1.pdf) и ничего не менять, кроме собственно кода, то запуск должен свестись к простому `./run_pseudo_scp.sh /путь/к/папке/с/кодом`.

Касательно самого кода:

* Важно правильно указать пути в функции main (см. [методичку](https://github.com/ShunkevichDV/wave_find_path_sc_memory/blob/master/%D0%9F%D0%9F%D0%B2%D0%98%D0%A1-1.pdf)):

```C++
params.repo_path = "/ostis/kb.bin";
params.config_file = "/ostis/config/sc-web.ini";
params.ext_path = "/ostis/sc-machine/bin/extensions";
```

* Стоит иметь ввиду, что запустить среду разработки в контейнере нельзя, так что дебажить придётся сиаутами.

Когда код отработает, он напечатает в консоль вывод и сам выйдет из контейнера.

После этого можно внести изменения в код и повторить шаг 2.

### Дополнительно

Скрипт clean.sh удаляет все оставшиеся контейнеры. Это почти полностью выполняется в начале run.sh, так что при каждом новом запуске всё лишнее будет удаляться, но в самом конце развлечений лучше сделать дополнительно:

```bash
./clean.sh
```

В run_pseudo_scp.sh очистка выполняется и в начале, и в конце, поэтому чистить вручную необязательно.

Также можно посмотреть содержимое kb в уже собранной и запущенной системе, например, чтобы убедиться, что в контейнер попали правильные файлы. Для этого пока контейнер запущен в новой вкладке:

```bash
./show_kb.sh
```

Все найденные проблемы и их возможные решения будут [тут](https://github.com/ARtoriouSs/docker-ims.ostis/blob/master/troubleshooting.md "Расстрел проблем").

## Contributing

С точки зрения правильной организации docker контейнеров этот репозиторий далеко не в лучшем состоянии. Так вышло потому, что делал я это когда только начинал изучать докер, да и сама система довольно специфична. Сейчас у меня особо нет времени и мотивации заниматься рефакторингом, поэтому если кто-то хочет внести свой вклад в оптимизацию, то буду рад принять пулл реквест. Кстати, коммиты сюда можно включать в курсач :)

Актуальные проблемы описаны в [открытых issues](https://github.com/ARtoriouSs/docker-ims.ostis/issues), тут же можно предложить свои идеи или замечания.

**P.S.** Идейный вдохновитель [Маша](https://github.com/idealasgas "GitHub Маши") :)
