Вопрос: Как отделить конфигурацию от основного приложения? Это нужно, чтобы нам не приходилось создавать новый docker image для каждой среды, а использовать один и тот же docker image и изменять только конфигурации, которые располагаются отдельно. 
![[Pasted image 20241202135211.png]]
В Spring Boot есть решения этих 3 челленджей:
![[Pasted image 20241202135350.png]]
### Как SpringBoot приложение конфигурирует приложение?
Также снизу через 
- представлены способы, которыми мы можем конфигурировать наше SpringBoot приложение. Причем они расположены в порядке от менее приоритетного до более приоритетного.
![[Pasted image 20241202140134.png]]
###  Способы получить значения
![[Pasted image 20241202140928.png]]
Чтобы работало @ConfigurationProperties:
![[Pasted image 20241202145053.png]]
![[Pasted image 20241202145131.png]]
![[Pasted image 20241202145147.png]]
### SpringBoot Profiles
Хорошо, мы научились читать в нашем приложении данные из application.yml, а также из environment, но как изменять значения под разными environments.
С помощью профайлов мы можем выбирать, какой файл с properties будет загружен в наше приложении, а также какие бины будут созданы.

Для создания properties файлов для разных профайлов используется конвенция:
application_prod - чтобы этот файлик начал влиять на наше приложении, нужно, чтобы оно было запущено с включенным профайлом prod.
Дефолтный профайл работает по дефолту с application.properties.
![[Pasted image 20241202150944.png]]
Таким образом, чтобы создать профайл:
- application_< profile_name>.properties. Причем таких профайлов в нашем приложении мы создаем сразу под все среды.
Активировать профайл:
- spring.profiles.active=< profile_name>

Важно, что в профайлах, которые мы создаем должны находиться только специфичные настройки для среды. Т.е. все общее для всех профайлов и нам не нужно менять это свойство среди множества сред мы оставляем в application.yml, т.к. этот файлик применяется по умолчанию, а в других мы уже переписываем значения, которые хотим, например, datarouse.url.

Также в файлике мы задаем следующие настройки, чтобы файл был активирован, когда включает профайл с именем "qa". Это нужно, поскольку в имени профайла мы указали нижнее подчеркивание. Если бы мы указали application-qa, то Spring по дефолту, основываясь на имени файла, использовал бы наш application-qa файл, если qa профайл активирован.
![[Pasted image 20241202152617.png]]
![[Pasted image 20241202153201.png]]
Также мы должны указать в application.yml следующее, чтобы сообщить Spring о наших профайлах. Ноо! Это все нужно только в том случае, если мы не хотим использовать стандартную конвенцию имен профайлов - application-< profile_name>.yml. Если мы используем такую конвекнцию, то не нужно указывать ни on-profile, ни spring.config.import
![[Pasted image 20241202153715.png]]

Для активации профайла qa в application.yml файл пишем:
spring
	profiles:
		active:
		  -qa
![[Pasted image 20241202154703.png]]

Но мы теперь столкнемся с проблемой, что для того, чтобы активировать профайл, нужно изменять application.yml под каждую среду. Но это неправильно, т.к. мы хотим действовать в соотвествии с [[15 Factor]]. Поэтому мы должны уметь откуда-то снаружи при старте нашего приложения изменять значения нужных нам значений в application.yml файле.

### Как изменять значения в application.yml при старте приложения?

Мы можем использовать параметры командной строки при запуске jar файла. Причем параметры командной строки являются наивысшими по приоритету и поэтому без проблем заменят значения в application.yml. Помимо переписывания можем задавать и новые значения, которые не было в application.yml
![[Pasted image 20241202155253.png]]
Чтобы активировать профайл: --spring.profiles.active=prod

Мы можем использовать JVM properties. Они по приоритету ниже, чем передачу через параметры командной строки. Но выше, чем в application.yml
![[Pasted image 20241202155519.png]]
Чтобы активировать профайл: -Dspring.profiles.active=prod

Использовать environment variables. Их плюс в том, что они подходят и в том случае, если мы не используем Java, а какой-нибудь микросервис на другом языке. Только нужно соблюдать конвенцию: build.version в application.yml превращается в BUILD_VERSION, когда мы используем environment variables, чтобы переписать это значение.
![[Pasted image 20241202155835.png]]
Чтобы активировать профайл: SPRING_PROFILES_ACTIVE=prod.
Если хотим задать несколько значений, то используем ; между ними.

### Обсудим минусы предоставления значений способами, описанными выше
![[Pasted image 20241202165221.png]]
- Нет возможности закодировать пароль, чтобы не показывать его нигде в открытом виде. Ведь даже если мы передаем пароль через environment variables, то этот пароль видят те, кто занимаются разворачиванием приложения.
- Нет возможности изменить значения у уже запущенного микросервиса. Чтобы обновить значение, нужно перезапустить микросервис.
- Нет возможности версионирования. Когда мы переходим на новую версию приложения, то мы можем менять application.properties какие-то составляющие, значения и в таком случае, нам нужно, чтобы была возможность версионирвания. Также, неплохо было бы иметь возможность аудита.
- Даже если мы будем использовать автоматизацию для запуска приложений, то могут просочиться ошибки, ведь автоматизацию также пишут люди.

### Для решения этих проблем существует Spring Cloud Config Server
Это отдельное Spring приложение. К нему регистрируются микросервисы и он выдает им конфигурации. Т.е. это такое централизованное хранилище конфигураций. Причем место, где хранить эти конфигурации мы выбираем сами. Это может быть как github, так и какое-то локальное хранилище или еще где-то.
![[Pasted image 20241202170627.png]]
Spring Cloud - это набор модулей, которые помогают решить проблемы, связанные с реализацией Cloud applications. 
![[Pasted image 20241202170859.png]]

### Реализация
Нужно добавить зависимость Spring
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-config-server</artifactId>  
</dependency>
```
Также использовать аннотацию @EnableConfigServer
![[Pasted image 20241202174618.png]]
Теперь нужно где-то хранить все конфигурации, которые будет читать для нас ConfigurationServer:
- GitHub - самый популярный подход.
- classpath - указываем где хранится
- filesystem - где-то храним директорию с конфигурациями.

### Classpath способ
В проекте с Server Config в resource директории создаем .yml файлы. Но какое имя им дать? Для имени используем имя микросервиса, которому принадлежат файлы. Например, если мы храним в файле проперти микросервиса - accounts, то и файл называем accounts.yml. Имя файла должно совпадать с именем spring boot приложения, которому принадлежит файл - spring.application.name. 
Т.е. в application.yml в проекте accounts должно быть spring.application.name=accounts. А в Config Server проекте будут лежать файлы, которые связаны с accounts проектом с именами accounts.yml, accounts_qa.yml, accounts_prod.yml. И для других проектов аналогично.
![[Pasted image 20241202181547.png]]
Внутри файлов оставляем только информацию, которая свойствена этому профайлу. Т.е. то, что должно поменяться в зависимости от профайла.

Вот, храним только какие-то специфичные штуки, которые будут меняться.
account.yml:
![[Pasted image 20241202180521.png]]
accounts-prod.yml:
![[Pasted image 20241202180600.png]]
accounts-qa.yml:
![[Pasted image 20241202180626.png]]

Далее мы в Server Configuration классе в его application.yml прописываем:
![[Pasted image 20241202181247.png]]

И теперь мы можем проверить, работает или нет. Для этого обращаемся к серверу, например, по http://localhost:8071/accounts/prod
И получим как раз содержимое файлика accounts-prod.yml
![[Pasted image 20241202181751.png]]
Вместе с prod переданы еще и default пропертис. Но это потому что они загружаются всегда по дефолту, а потом уже будут переписаны теми, что указаны в accounts-prod.yml

### Подключаем наш микросервис к серверу конфигураций
Теперь научимся подключать к нашему микросервису эти файлы конфигурации, которые располагаются на Configuration Server.

Удаляем все application-< profile>.yml файлы. Они теперь не нужны, т.к. будем получать их с сервера.
Дефолтный application.yml оставляем и не забываем указать в нем имя нашего приложения, т.к. именно по этому имени он будет отдавать нашему микросервису его конфигурационный файл. 

Добавляем зависимость client в наш микросервис:
```xml
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
```

```xml
 <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```

```xml
<properties>
    <java.version>17</java.version>
    <spring-cloud.version>2023.0.3</spring-cloud.version>
  </properties>
```

В application.yml:
```yml
spring:
  config:
    import: "optional:configserver:http://localhost:8071/"
```
optional - указывает на то, что если по каким-либо причинам config server не ответил, то мы продолжаем запускать наше приложение, но, просто не обновим значения пропертей, которые обновились бы, при использовании выбранного профайла. 
configserver - говорит спрингу, что мы будем использовать config server для конфигураций, а потом указывает хост и порт, на котором работает этот config server. После этого мы запускаем приложении и можем активировать любой из профайлов теми способами, которые описывали ранее(JVM, program variables, environment variables). И все будет работать, данные из файлика на сервере подтянутся в соотвествии с активированным профайлом.
Также мы можем активировать дефолтный профайл при запуске приложения:
![[Pasted image 20241202184247.png]]

### Вариант хранения конфигураций в file system
Почему этот подход есть смысл использовать? Потому что мы можем сохранить все проперти в какую-то защищенную директорию, которую никто не может читать, кроме Config Server.
Для этого нужно просто в application.yml для Configuration Server указать file:/// и дальше полный путь до директории, в которой будут находиться все application.yml файлы для каждого из наших микросервисов.  (Это относительно прошлого подхода. Т.е. мы еще оставляем настроку spring.profiles.active = native как это было в прошлом варианте хранение пропертей)
![[Pasted image 20241203130827.png]]
Содержимое директории config:
![[Pasted image 20241203130922.png]]
### Reading Configuration from a GitHub
Наиболее предпочтительный вариант, поскольку можно также настроить его приватность, есть аудит, т.е. видно, кто и когда менял эти конфигурационные файлы. Ну и само собой версионирование, т.е. сможем посмотреть всю историю изменений этих файлов по коммитам.
Создаем GitHub репозиторий(пока рассмотрим вариант с публичным, т.к. для приватного нужно дополнительно уметь аутентифицировать наш config server на гитхаб).
В application.yml файле Config Server указывем следующие настройки:
```yml
spring:  
  application:  
    name: "configserver"  
  profiles:  
    # active: native - для classpath и filesystem варианта  
    active: git  
  cloud:  
    config:  
      server:  
        git:  
          uri: "https://github.com/eazybytes/eazybytes-config.git"   
# native:  
          #search-locations: "file:///home//user//Documents//config"          default-label: main # указываем branch  
          timeout: 5 # ждем максимум 5 секунд  
          clone-on-start: true # клонируем репозиторий при старте config server  
          force-pull: true # после клонирование реп-я мы можем изменить что-либо локально  
          # этой настройком мы говорим, чтобы все эти изменения перезаписывались теми, что расположены на GitHub        # search-locations: "classpath:/config"
```
### Encryption & Decryption of properties inside Config server
Мы хотим, чтобы на GitHub properties хранились в зашифрованном виде.
В application.yml ConfigServer указываем secret-key, который будет использоваться для шифрования. 
![[Pasted image 20241203134258.png]]
email зашифровать можно через config server, передав в body текст, который нужно зашифровать по url: localhost:8071/encrypt POST запрос. И вернется строка с зашифрованным текстом.
![[Pasted image 20241203134534.png]]

После этого на github мы можем хранить информацию в зашифрованном виде, а для клиентов(микросервисов, которые будут обращаться к Config Server, чтобы получить проперти) он будет расшифровывать автоматически.
Также есть возможность расшифровать строку через Config Server:
![[Pasted image 20241203134947.png]]
### Refresh configurations at runtime using refresh actuator path
Мы хотим иметь возможность изменять конфигурации и чтобы микросервис, который использует эту конфигурацию не нужно было перезагружать, а эти изменения применились автоматически.

Сейчас мы разбираем именно вариант с использованием Spring Actuator. Поэтому такая зависимость должна находиться в нашем приложении.
После чего мы включаем его и открываем все endpoint-ы Spring Actuator(это мы и сделали через \* . Вместо нее могли написать какие-то конкретные эндпоинты). Такая настройка должна быть во всех микросервисах в файле application.yml:
![[Pasted image 20241203140233.png]]

При этом на Config Server у нас никаких проблем нет. Если мы на гите обновим файлы, то Config Server узнает об этом, т.е. он не будет полагаться на кэш, а будет ходить на гит для получения файлов, но с микросервисами так не работает, они уже запущены и работают с конкретным файлом с пропертями, который получили при старте. Поэтому для них нужны дополнительные шаги в вид использования Spring Actuator, который мы подключили и настроили на шаге выше.

После чего мы просто посылаем POST запрос на localhost:8080/actuator/refresh. После этого property значения поменяются на новые из Config Server.
![[Pasted image 20241203141507.png]]
Но у такого подхода есть проблема - нужно отправлять POST запросы для каждого микросервиса, чтобы обновить проперти. А руками это делать для сотни микросервисов такое себе(конечно, мы можем написать скрипты, которые автоматом сделают эти запросы).

### Refresh configurations using Spring Cloud Bus
С помощью этого подхода мы решим проблему с тем, что нужно вручную отправить POST refresh на actuator каждого микросервиса. Для этого можно использовать Spring Cloud Bus, которая работает как легковесный брокер поверх kafka или rabbitmq. 
Мы сможем отправить один  POST /refresh запрос и Spring Cloud Bus забродкастит все эти изменения всем микросервисам.
Подключаем зависимость во все проекты(config server тоже)
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>  
</dependency>
```
Также открываем все эндпоинты актуатора, как мы делали это выше.

Настраиваем каждый микросервис, чтобы он мог работать с rabbitmq(username и password используем дефолтные для rabbitmq)
spring.rabbitmq:
![[Pasted image 20241203144612.png]]
После этого для рефреша конфигураций всех микросервисов используем:
localhost:8080/actuator/busrefresh
![[Pasted image 20241203145107.png]]

![[Pasted image 20241203145613.png]]

Минусом является то, что нужно все равно сделать 1 post запрос для обновления. Это в принципе не проблема, если мы не планируем часто менять application.properties файлы. Но если мы хотим менять их довольно часто, то нужно использовать еще один подход.
### Refresh config at runtime using Spring Cloud Bus & Spring Cloud Config monitor
Если интересно, то посмотритшь на udemy в уроке 89.
### Docker Compose with Configuration Server
Теперь нам нужно научиться разворачивать это все в Docker Compose. Причем для каждой среды нужно будет написать свой Docker compose. Т.е. для prod - один, для qa - другой.
### Liveness and Readiness probes(проверки)
Нужны для того, чтобы уметь определять в каком состоянии находится наш контейнер. И в случае, если с контейнером что-то не то, то мы могли начать выполнять какие-то шаги по решению проблемы. Например, Kubernetes будет использовать их, чтобы проверить, что тот контейнер, который он поднял реально работает, а если не работает, то удалить и попробовать поднять еще раз.

liveness prob - это когда мы отправляем сигналы проверки того, что с контейнером все хорошо и он нам отвечает, все хорошо у него или нет. Простыми словами: такая проверка отвечает на вопрос жив контейнер или нет -true false. 

readiness prob - проверка того, готов ли контейнер начать получать трафик из сети. Т.е. может быть такое состояние, что контейнер жив, т.е. liveness prob вернуло true, но вот принимать трафик он не может. Это частая практика, когда контейнер только запустился, но еще не в состоянии принимать запросы, например, потом что происходит разогрев кэшей или еще какой-нибудь background task происходит.

![[Pasted image 20241203154847.png]]

Spring Boot Actuator имеет endpoints, которые позволяют как раз оценить liveness и readiness приложения. /actuator/health/livenss и /actuator/health/readiness. Или вернется все вместе /actuator/health

В Config Server в application.yml добавляем:
```yml
management:  
  endpoints:  
    web:  
      exposure:  
        include: "*"  
  health: # просим actuator включить информацию о readiness и liveness  
    readinessstate:  
      enabled: true  
    livenessstate:  
      enabled: true  
  endpoint:  
    health:  
      probes:  
        enabled: true
```
После этого можем обращаться и получать такие ответы:
![[Pasted image 20241203155715.png]]
UP = true.

Вот такой docker-compose файл мы получили для default профайла:
```yml
services:  
  rabbit:  
    image: rabbitmq:3.13-management  
    hostname: rabbitmq  
    ports:  
      - "5672:5672"  
      - "15672:15672"  
    healthcheck:  
      test: rabbitmq-diagnostics check_port_connectivity  
      interval: 10s  
      timeout: 5s  
      retries: 10  
      start_period: 5s  
    networks:  
	  - eazybank
  configserver:  
    image: "eazybytes/configserver:s6"  
    container_name: accounts-ms  
    ports:  
      - "8071:8071"  
    depends_on:  
      rabbit:  
        condition: service_healthy  
    healthcheck:  # test - команда, которой проведем проверку 
      test: "curl --fail --silent localhost:8071/actuator/health/readiness | grep UP || exit 1"  
      interval: 10s # интервал между попытками  
      timeout: 5s # время на выполнение команды  
      retries: 10 # 10 попыток выполнить эту команду  
      start_period: 10s # выполнить команду из test: через 10 секунд  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - eazybank  
  accounts:  
    image: "eazybytes/accounts:s6"  
    container_name: accounts-ms  
    ports:  
      - "8080:8080"  
    depends_on:  
      configserver:  
        condition: service_healthy # accounts будет ждать, пока configserver не # пройдет проверку на healthcheck  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - eazybank  
    environment:  
      SPRING_APPLICATION_NAME: "accounts"  
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"  
      SPRING_PROFILES_ACTIVE: default  
  loans:  
    image: "eazybytes/loans:s6"  
    container_name: loans-ms  
    ports:  
      - "8090:8090"  
    depends_on:  
      configserver:  
        condition: service_healthy  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - eazybank  
    environment:  
      SPRING_APPLICATION_NAME: "loans"  
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"  
      SPRING_PROFILES_ACTIVE: default  
  cards:  
    image: "eazybytes/cards:s6"  
    container_name: cards-ms  
    ports:  
      - "9000:9000"  
    depends_on:  
      configserver:  
        condition: service_healthy  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
    networks:  
      - eazybank  
    environment:  
      SPRING_APPLICATION_NAME: "cards"  
      SPRING_CONFIG_IMPORT: "configserver:http://configserver:8071/"  
      SPRING_PROFILES_ACTIVE: default  
networks:  
  eazybank:  
    driver: "bridge"
```

Но мы видим, что тут много повторяющегося текста.
В Docker Compose оказывается есть механизм наследования.
Мы создаем файлик:
![[Pasted image 20241203163506.png]]

В нем пишем повторяющуюся информацию и также используем наследование. common-config.yml:
```yml
services:
  network-deploy-service:
    networks:
      - eazybank

  microservice-base-config: # отнаследовались от прошлого и теперь внутри есть 
  # networks: eazybank
    extends:
      service: network-deploy-service
    deploy:
      resources:
        limits:
          memory: 700m
    environment:
      SPRING_RABBITMQ_HOST: "rabbit" # имя service в котором запущен rabbitmq

  microservice-configserver-config:
    extends:
      service: microservice-base-config
    environment:
      SPRING_PROFILES_ACTIVE: default
      SPRING_CONFIG_IMPORT: configserver:http://configserver:8071/
```
А теперь в docker-compose также пишем extends, указываем файл и имя сервиса из которого хотим забрать часть настроек. Благодаря этому мы можем изменять настройки в одном файле и переиспользовать их во множестве docker-compose файлах и микросервисах, оставляя в docker-compose только уникальные значения.
```yml
services:
  rabbit:
    image: rabbitmq:3.13-management
    hostname: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    healthcheck:
      test: rabbitmq-diagnostics check_port_connectivity
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 5s
    extends:
      file: common-config.yml
      service: network-deploy-service

  configserver:
    image: "eazybytes/configserver:s6"
    container_name: configserver-ms
    ports:
      - "8071:8071"
    depends_on:
      rabbit:
        condition: service_healthy
    healthcheck:
      test: "curl --fail --silent localhost:8071/actuator/health/readiness | grep UP || exit 1"
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 10s
    extends:
      file: common-config.yml
      service: microservice-base-config

  accounts:
    image: "eazybytes/accounts:s6"
    container_name: accounts-ms
    ports:
      - "8080:8080"
    depends_on:
      configserver:
        condition: service_healthy
    environment:
      SPRING_APPLICATION_NAME: "accounts"
    extends:
      file: common-config.yml
      service: microservice-configserver-config

  loans:
    image: "eazybytes/loans:s6"
    container_name: loans-ms
    ports:
      - "8090:8090"
    depends_on:
      configserver:
        condition: service_healthy
    environment:
      SPRING_APPLICATION_NAME: "loans"
    extends:
      file: common-config.yml
      service: microservice-configserver-config

  cards:
    image: "eazybytes/cards:s6"
    container_name: cards-ms
    ports:
      - "9000:9000"
    depends_on:
      configserver:
        condition: service_healthy
    environment:
      SPRING_APPLICATION_NAME: "cards"
    extends:
      file: common-config.yml
      service: microservice-configserver-config

networks:
  eazybank:
    driver: "bridge"
```
### Как теперь распространить все это на другие среды?
Для этого мы копируем эти файлы в соотвествующие директории
![[Pasted image 20241203170120.png]]
И дальше нужно всего-то в common-config поменять нужно просто поменять одну настройку, чтобы все контейнеры использовали environment переменную, с помощью которой и запустились для другого энвайрмента.
![[Pasted image 20241203170156.png]]
