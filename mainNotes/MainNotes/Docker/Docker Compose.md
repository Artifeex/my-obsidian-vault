Позволяет запускать сразу несколько контейнеров, используя 1 команду. Все эти запущенные контейнеры также можно остановить или удалить с помощью одной команды.

![[Pasted image 20241010204315.png]]
Самое вкусное DNS - т.е. мы сможем обращаться по имени к какому-нибудь контейнеру, т.е. например, GET запрос на url и в нем указывать в качестве ip адреса имя контейнера. И тогда DNS зарезолвит имя контейнера на его адрес.

![[Pasted image 20241010210259.png]]
Для каждого сервиса будет создан свой контейнер. Мы указываем название сервисов и внутри сервисов уже указываем какие-то их настройки.
Например, для сервиса mongo мы говорим, что хотим, чтобы использоваться image с именем mongo. Т.е. теперь, когда будет создаваться контейнер mongo, то он будет искать image с именем mongo, если не найдет локально, то пойдет искать на docker hub. Скачаем этот образ и создаст контейнер.

В сервисе с именем app же мы используем build и указываем путь до dockerfile. Т.е. здесь мы говорим, что нужно будет создать контейнер, используя наш кастомный образ. Причем сам кастомный образ также нужно создать, а чтобы это сделать, нужно использовать docker build и указать директорию, в которой будет находиться Dockerfile.

### Создадим main.py
В котором будем подключаться к БД. Причем стоит заметить, что есть модуль pymongo, которого нет в стандартной библиотеке. Его мы установим при запуске контейнера с нашим приложением.(В случае Java можно собрать толстый jar, в котором будут все зависимости, наверное, так и делают, т.к. мы же собираемся потом просто запустить контейнер, а не устаналивать какие-то доп зависимости локально).
![[Pasted image 20241010211351.png]]
Теперь пишем сам Dockerfile:
![[Pasted image 20241010213056.png]]

Теперь нужно создать файл с именем docker-compose.yml и сконфигурировать его. Структура проекта сейчас вот такая:
![[Pasted image 20241010213620.png]]
Сам docker-compose файл. Объяснения того, что происходит выше. Но стоит заметить, что мы не настраивали никакой порт маппинг. Т.е. ни контейнер app, ни контейнер mongo не будут доступны из вне ни по какому порту. Но при этом между собой данные контейнеры могут взаимодействовать(из main.py мы подключаемся к mongo, используя URL, в котором указываем название сервиса. А это название смаппится на уже IP контейнера с mongo). 
![[Pasted image 20241010213847.png]]

### Запуск docker-compose
Теперь можем запустить, для этого вводим команду
`docker-compose up`, находясь в папке, в которой лежит docker-compose.yml
![[Pasted image 20241010214355.png]]

После этого создается сеть внутри докер. Сеть нужна для изолированного взаимодействия контейнеров внутри этой сети. Далее создается image для app на основе Dockerfile, который мы написали. Потом скачивается образ для mongo из dockerHub. После этого создаются 2 контейнера, один для app, другой для mongo. И эти два контейнера добавляются в docker сеть, которая была создана в самом начале:
![[Pasted image 20241010214653.png]]

### Для запуска контейнеров в фоне используется -d
`docker-compose -d up`
После этого поднимется 2 контейнера. Один с app, другой с mongo. Причем app завершит работу, т.к. мы в нем просто подключились и вывели все БД и закончили работа. А mongo продолжит работу и при вызове `docker ps` мы увидим, что контейнер с mongo будет активным.

### Остановка всех контейнеров внутри docker-compose
`docker-compose down` - остановит все контейнеры, которые запустили ранее при помощи up, а также удалит остановленные контейнеры и созданную сеть.
![[Pasted image 20241010215811.png]]

### Пересоздание образов
Когда мы изменили как-то наше приложение, то нужно заново создавать image. Чтобы при запуске image создали заново используем опцию --build. Но там будет использован cache, поэтому если изменили только какой-то исполняемый файлик, то просто выполнится COPY из Dockerfile, а для всего остального используется кэш.
![[Pasted image 20241010220245.png]]