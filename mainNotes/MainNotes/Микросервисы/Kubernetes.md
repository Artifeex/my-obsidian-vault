### Какие задачи будем решать?
![[Pasted image 20241218093110.png]]
1. Автоматический деплой, обновление версий приложения без downtime, а также возможность выполнить rollback до старой версии, в случае ошибок.
2. Как убивать контейнеры, которые не отвечают. Как проверять жив контейнер или нет. А также пускать в контейнер трафик только в том случае, если он полностью готов. Как перезапускать контейнеры, которые не отвечают?
3. Возможность автоматического автоскеллинга в том случае, если нагрузка повысилась

Все эти задачи решает kubernetes - опен сорсный фреймворк для оркестрирования контейнеров.

Кубернетес является нейтральным к облакам. Т.е. он работает схожим образом как для ситуации, когда мы развернули его локально, так и в ситуации, когда мы развернули его в aws.

Kubernetes предлагает следующие функии:
![[Pasted image 20241218094240.png]]
Service discovery and load balancing. Kubernetes возьмет на себя load balancing - тогда мы перейдем от client-side balancing, который предлагает нам eureka server к server-side balancing, который предлагает нам kubernetes.

### Внутренняя архитектура k8s
![[Pasted image 20241218094938.png]]
Когда мы говорим о kubernetes - мы говорим о кластере. Кластер - это множество серверов, которые работают вместе.

Но почему мы не можем деплоить микросервисы, используя docker compose? Потому что с помощью docker-compose мы можем задеплоить все приложения только на одной машине, на которой и поднимаем микросервисы. Но в реальных приложениях у нас может быть 100-ни микросервисов, и они не могут быть запущены на одном сервере.
Вместо этого мы хотим деплоить наши микросервисы на разных серверах - нодах внутри кластера. Помимо этого docker-compose не может предоставить автоскеллинг, rollout, rollback.

В k8s кластере есть два типа node:
- master node - нужна для того, чтобы контролировать весь k8s кластер. 
  Внутри нее есть компонент - kube API server. Он предоставляет API с помощью которого извне можно будет взаимодействовать с k8s. Есть два варианта - использовать web Admin UI или же использовать консоль - kubectl CLI. Когда мы будем выполнять какие-то команды в консоли или через админку, то их будет принимать kube api server. После прочтения этих инструкций kube api server передаст их в sheduler. Sheduler читает требования(инструкции), переданные ему от kube api server и, например, если была инструкция по деплою, то он определит, на какой ноде выполнить деплой, используя различные метрики о том, под какой нагрузкой находятся ноды. Дальше он эту информацию передает обратно в kube api server и он уже выполняет деплой на той ноде, которую определил sheduler.
  
  Contol Manager - ответственен за то, чтобы следить за состоянием контейнеров и worker нод. Если будет найдена какая-то проблема, то он также ответственен за замену не работающей ноды или контейнера. Простыми словами - мы передаем в него желаемое состояние нашего приложения - например, должно быть 3 микросервиса accounts. И он следит, чтобы в кластере было 3 микросервиса accounts и в случае возникновения проблем, он будет перезапускать контейнеры или создать новые, чтобы удовлетворять желаемому состоянию, которое передается пользователем.
  
  etcd - key-value хранилище информации о нашем кластере. Например, если control manager в какой-то момент времени будет иметь вопрос, а сколько реплик accounts микросервиса он должен поддерживать, то он будет обращатсья в etcd для получения этой информации. И также если пользователь отправил инструкции в kube api server, то он должен сохранить эту информацию в etcd, чтобы потом control manager, sheduler могли эту информацию получить, если потребуется.
  
- worker nodes - на них работают контейнеры.
  kubelet - компонент, который запущен на каждой worker node. Именно с ним общается kube api server для передачи инструкций в worker ноду. 
  
  container runtime - environment для запуска и работы контейнеров. Чаще всего docker.
  
  pod - минимальный deploy unit. Контейнеры работают не напрямую в ноде, а в подах. Поды нужны для изоляции приложений. Чаще всего в одном поде - один контейнер. Но если есть основной контейнер и еще какой-то побочный, которые выполняет вспомогательную функцию, например, логирование, то они могут быть запущены в одном поде. 
  
  kube-proxy - для сетевого общения. Т.е. kube-proxy предоставляет внешний доступ к подам. Но также и может наоборот заблокировать весь возможный внешний трафик, разрешая общение только внутри кластера.
![[Pasted image 20241218110215.png]]
![[Pasted image 20241218110303.png]]

Docker Desktop может нам установить k8s локально. Но в таком случае будет создана только 1 нода. После установки можем пользоваться командами.

#### Получаем кластеры запущенные на компике
**![[Pasted image 20241218113806.png]]**
#### Получить ноды запущенные на компике
`kubectl get nodes`

#### Helm - это package manager для Kubernetes. С помощь. него мы сможем устанавливать различные компоненты. Например, чтобы добавить Dashboard(веб-сайт) мы также будем использовать helm
Но перед использованием helm мы должны установить его.

Вот так с помощью helm можно установить dashboard:
![[Pasted image 20241218123547.png]]
![[Pasted image 20241218123621.png]]
![[Pasted image 20241218123720.png]]
По такому URL будет доступен dashboard.
![[Pasted image 20241218123910.png]]

Первое, что нужно сделать - это создать Service Account. Он нужен для того, чтобы создать пользователя и наделить его правами и тогда мы сможем заходить в dashboard под этим пользователем:
![[Pasted image 20241218124420.png]]
kind - тип объекта, который мы хотим создать.
metadata - метаданные, которые мы передаем для создания ServiceAccount.
name - название ServiceAccount
namespace - название namespace, в котором будет создан этот объект. Если не указывать - будет дефолтный. namespaces нужны для изоляции и группировки. У нас есть QA, DEV, PROD environments, тут для тех же целей namespaces.
![[Pasted image 20241218124228.png]]
После того, как создали .yml файлик с конфигурацией выше мы должны сказать k8s выполнить данный файл:
![[Pasted image 20241218125911.png]]

Мы создали ServiceAccounts, но теперь нам нужно дать ему привилегии. Для этого используется ClusterRoleBinding.
![[Pasted image 20241218130311.png]]
Также применяем этот файлик с помощью kubectl apply -f

Даьше генерируем токен и заходио по нему в dashboard:
![[Pasted image 20241218141116.png]]

Можно также создать токен, который будет долгоживущим. Иначе придется каждый раз генерировать новый токен, когда dashboard попросит ввести новый.

### Задеплоим configserver в kubernertes
Начинаем именно с него, поскольку все остальные микросервисы зависят от этого микросервиса.
Создаем файл configserver.yml
![[Pasted image 20241219100605.png]]

