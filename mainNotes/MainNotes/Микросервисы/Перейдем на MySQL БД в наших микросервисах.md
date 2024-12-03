Обычно используется подход с одной БД для каждого микросервиса. Будем использовать докер и запускать так БД. Вот такой командной запустим БД для accountsdb:
`docker run -p 3306:3306 --name accountsdb -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=accountsdb -d mysql`
В микросервисах указываем параметры подключения:
![[Pasted image 20241203181916.png]]

Чтобы все работало в docker-compose.
Это файл common-config.yml, куда мы выделяем повторяющийся код:
```yml
services:  
  network-deploy-service:  
    networks:  
      - eazybank  
  
  microservice-db-config:  
    extends:  
      service: network-deploy-service  
    image: mysql  
    healthcheck:  
      test: [ "CMD", "mysqladmin" ,"ping", "-h", "localhost" ]  
      timeout: 10s  
      retries: 10  
      interval: 10s  
      start_period: 10s  
    environment:  
      MYSQL_ROOT_PASSWORD: root  
  
  microservice-base-config:  
    extends:  
      service: network-deploy-service  
    deploy:  
      resources:  
        limits:  
          memory: 700m  
  
  microservice-configserver-config:  
    extends:  
      service: microservice-base-config  
    environment:  
      SPRING_PROFILES_ACTIVE: default  
      SPRING_CONFIG_IMPORT: configserver:http://configserver:8071/  
      SPRING_DATASOURCE_USERNAME: root  
      SPRING_DATASOURCE_PASSWORD: root
```

А это уже docker-compose.yml файл. Важно заметить, что для БД, также в идеале нужно писать healthcheck, т.к. наш микросервис, который использует эту БД должен подключиться к ней, поэтому она должна быть готова к принятию запросов.

Также важно заметить, что когда мы пишем SPRING_DATASOURCE_URL: "jdbc:mysql://cardsdb:3306/cardsdb"  то мы указываем порт БД внутри докера, а не тот порт, который контейнер паблишит наружу. Это потому что все контейнеры работают внутри одной docker сети.
```yml
services:  
  accountsdb:  
    container_name: accountsdb  
    ports:  
      - 3306:3306  
    environment:  
      MYSQL_DATABASE: accountsdb  
    extends:  
      file: common-config.yml  
      service: microservice-db-config  
  
  loansdb:  
    container_name: loansdb  
    ports:  
      - 3307:3306  
    environment:  
      MYSQL_DATABASE: loansdb  
    extends:  
      file: common-config.yml  
      service: microservice-db-config  
  
  cardsdb:  
    container_name: cardsdb  
    ports:  
      - 3308:3306  
    environment:  
      MYSQL_DATABASE: cardsdb  
    extends:  
      file: common-config.yml  
      service: microservice-db-config  
  
  configserver:  
    image: "eazybytes/configserver:s7"  
    container_name: configserver-ms  
    ports:  
      - "8071:8071"  
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
    image: "eazybytes/accounts:s7"  
    container_name: accounts-ms  
    ports:  
      - "8080:8080"  
    environment:  
      SPRING_APPLICATION_NAME: "accounts"  
      SPRING_DATASOURCE_URL: "jdbc:mysql://accountsdb:3306/accountsdb"  
    depends_on:  
      accountsdb:  
        condition: service_healthy  
      configserver:  
        condition: service_healthy  
    extends:  
      file: common-config.yml  
      service: microservice-configserver-config  
  
  loans:  
    image: "eazybytes/loans:s7"  
    container_name: loans-ms  
    ports:  
      - "8090:8090"  
    environment:  
      SPRING_APPLICATION_NAME: "loans"  
      SPRING_DATASOURCE_URL: "jdbc:mysql://loansdb:3306/loansdb"  
    depends_on:  
      loansdb:  
        condition: service_healthy  
      configserver:  
        condition: service_healthy  
    extends:  
      file: common-config.yml  
      service: microservice-configserver-config  
  
  cards:  
    image: "eazybytes/cards:s7"  
    container_name: cards-ms  
    ports:  
      - "9000:9000"  
    environment:  
      SPRING_APPLICATION_NAME: "cards"  
      SPRING_DATASOURCE_URL: "jdbc:mysql://cardsdb:3306/cardsdb"  
    depends_on:  
      cardsdb:  
        condition: service_healthy  
      configserver:  
        condition: service_healthy  
    extends:  
      file: common-config.yml  
      service: microservice-configserver-config  
  
networks:  
  eazybank:  
    driver: "bridge"
```