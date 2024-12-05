Нам нужна единая точка входа, в которую будут обращаться клиенты для общения с микросервисами. Зачем? А потому что иначе как клиенты будут узнавать об IP и портах новых микросервисов, которые будут подниматься динамически? Также на клиентов сразу переходит обязанность балансировку нагрузки между инстансами микросервиса, что тоже не круто. Также, если пользователи будут напрямую общаться с микросервисами, то в каждом из них нужно будет реализовывать проверку безопасности, аудита и т.д. Что приводит к дублированию кода и его неконсистености, т.к. та же проверка Security может быть по-разному написана в разных микросервисах. 
![[Pasted image 20241205115450.png]]
Все эти проблемы решаются Edge server(или еще его называют API Gateway).

### Что еще может делать Gateway в наших микросервисах:
![[Pasted image 20241205120314.png]]
### Spring Cloud Gateway
Для реализации такого Edge Server нам поможет Spring Cloud Gateway. Т.к. в него будет приходить очень много трафика, то он основан на Reactive Framework, чтобы избежать большого количество thread-ов. 
Одной из важных фич Gateway является то, что мы можем динамически направлять запросы в наши микросервисы. Т.е. мы можем писать логику, куда именно направится клиентйский запрос.
Например:
- В headere запроса может быть указана версия микросервиса, с которым клиент хочет пообщаться. Поэтому мы можем извлечь эту информацию и направить запрос в определенный сервис.
- [[Sticky session]] - это когда мы хотим, чтобы если к нам пришел запрос для которого уже существует сессия, то мы хотим, чтобы все такие запросы обрабатывались одним и тем же инстансом микросервиса. Т.е. мы будем перенаправлять запросы с определенной сессией в один и тот же микросервис благодаря Gateway.
- Можно также реализовать Ratelimiting, чтобы ограничить запросы от одного клиента.

### Архитектура Sping Cloud Gateway
![[Pasted image 20241205122125.png]]
1. Когда запрос приходит от клиента, то нужно определить, в какой микросервис обратился клиент и хочет попасть. Т.е. происходит анализирование запроса клиента. Тут нет никакой ИИ, мы реально руками напишем это перенаправление. Компонент, который этим занимается называется Gateway Handler Mapping.
2. После того, как с помощью Gateway Handler Mapping мы определили в какой микросервис нужно перенаправить запрос, то дальше запрос попадает в Predicates, которые проверяет запрос, используя conditions. По сути эти предикаты, которые что-то проверяют и возвращают true или false. И в случае false - значит запрос не удовлетворил какому-то предикату и дальше запрос не отправляется, а пользователю происходит ответ с ошибкой.
3. Если запрос прошел через предикаты, то он попадает в Pre Filters. Вот в них уже происходит большинство cross-cutting concerns. Т.е. Security checks, Auditing, Logging и т.д.
4. После Pre filters запрос наконец-то будет направлен в соотвествующий микросервис. 
5. Микросервис отправляет ответ и ответ попадает в Post Filter. В них мы также можем запихнуть какую-то доп логику обработки ответа(например, content-type). После этого ответ попадает в Gateway Handler Mapping и отправляет ответ клиенту.

Команда Spring Cloud реализовала для нас множество каких-то стандартных Pre Filters, которые покрывают базовые операции. И мы сможем использовать эти фильтры для своих нужд, а если стандартных не хватит, то написать свои. Вот список стандартных фильтров: 
![[Pasted image 20241205123436.png]] 
### Создание Spring Cloud Gatewat Project
Список зависимостей следующий:
![[Pasted image 20241205124612.png]]
application.yml:
![[Pasted image 20241205125728.png]]
Видим, что мы снова используем ConfigServer, поэтому остальные настройки находятся там:
Порт и подключение к eureka, т.к. Gateway будет клиентом eureka, чтобы получать информацию об адресах микросервисов. Но помимо параметров подключения к eureka мы должны указать Gateway, чтобы он использовал этот eureka сервер для извелчения информации(ведь по факту помимо eureka, могут существовать и другие discovery servers, с которых также можно было бы получать информацию)
![[Pasted image 20241205125811.png]]
В application.yml добавляем настройку. На самом деле cloud.gateway.discovery.locator.enabled=true включает дефолтный RouteLocator(об этом будет ниже)
![[Pasted image 20241205130308.png]]

Если мы обратимся по actuator/gateway/routes то получим следующий вывод:
![[Pasted image 20241205141113.png]]

![[Pasted image 20241205140847.png]]
Есть фильтр, который переписывает URL, который будет реально использоваться для обращение к микросервису. Т.е. у нас сработает предикат, например, который ожидает в URL увидеть ACCOUNTS и вернет true, дальше запрос отправится в фильтр, который из URL уберет слово ACCOUNTS, т.к. эта информация нужно только для gateway, чтобы обратиться к Eureka и получить по ACCOUNTS всю информацию(IP, порты и т.д.) А дальше уже, нужно передать оставшуюся часть URL, по которому изначально клиент и обращался. Поэтому и нужен rewrite url фильтр.

Стоит заметить, что пока в данном сетапе нашего приложения клиенты смогут напрямую обращаться к нашим микросервисам, но в будущем мы настроим все так, что к микросервисам можно будет обратиться только через gateway и никак иначе.

### Сделаем так, чтобы в URL при указании названия микросервиса мы могли использовать маленькие буквы
Для этого указываем:
![[Pasted image 20241205142038.png]]
### Реализуем кастомный routing в Spring Cloud Gateway
Т.е. мы хотим уметь задавать шаблон URL и определять, куда по такому URL будут перенаправлены запросы из gateway.
![[Pasted image 20241205142610.png]]

Для реализации мы должны создать bean RouteLocator. 
![[Pasted image 20241205144639.png]]
path - это путь для которого мы пишем routing. Т.е. когда придет запрос по такому пути, то мы его перенаправим. Но куда перенаправим? Тут уже мы делаем подход такой же, как это делает Spring Cloud. Мы переписываем URL, убирая из него информацию о том, к какому микросервису был произведен запрос(т.е. убрали /eazybank/accounts из запроса), и сохраняем в segment(это название группы в регулярном выражении) оставшуюся часть запроса, которая имеет смысл для конечного микросервиса. Т.е. тот URL, по которому мы бы могли напрямую обратиться к микросервису. 
И в uri указываем, куда направить этот запрос. lb - load balancer:ACCOUTNS(название микросервиса, по которому он зареган в eureka). Т.е. вот этот URI нужен только для того, чтобы направить запрос в определенный микросервис.

Давайте разберем синтаксис `(?<segment>.*)`.
### Общий синтаксис

`(?<segment>.*)` — это **регулярное выражение** (RegEx), используемое для указания группы с именем `segment`.  
Этот синтаксис определяется следующим образом:

1. **`(?<имя>...)`**:
    
    - Это именованная группа в регулярном выражении.
    - `segment` — имя группы.
    - Всё, что попадает под шаблон внутри скобок `(...)`, будет захвачено и сохранено с именем `segment`.
2. **`.*`**:
    
    - Это шаблон регулярного выражения, который означает "любой символ (`.`), повторяющийся ноль или более раз (`*`)".
    - Другими словами, это захватывает любую последовательность символов (включая пустую строку).

Но тот RouteLocator, который мы написали будет работать вместе с дефолтным, а мы этого не хотим, т.к. будут перенаправляться как /eazybank/accounts так и просто /accounts запросы. Мы же хотим, чтобы работали только /eazybank/accounts. Тогда мы ставим следующую настройку в false: 
![[Pasted image 20241205144901.png]]

Не забываем, что мы можем пользоваться дефолтными фильтрами, которые написаны командной Spring для нас. НАпример, мы хотим добавить Header в Response. Для этого уже есть готовы фильтр, поэтому мы можем добавить его.
![[Pasted image 20241205145930.png]]
После этого в response мы получим этот хедер, который мы добавили в фильтре
![[Pasted image 20241205145959.png]]

### Как написать кастомный фильтр
Задача - хотим, чтобы при новом запросе, попадающем в Gateway генерировался случайный ID запроса и потом этот ID передавался в хедере запроса при передаче его в другие микросервисы. И потом, когда пришло время отправлять ответ, то этот ID также отправлялся пользователю. Плюс добавим логирование, чтобы мы могли отследить, как перемещался запрос между микросервисами. И например, если произошла ошибка, клиенту вернулся ID запроса и потом мы можем по этому ID обратиться к системе логирования и посмотреть, как перемещался запрос и найти в каком микросервисе произошла ошибка. 

Т.е. мы таким образом, хотим сделать tracing запросов.

```java
@Order(1) // Задаем порядок выполнения кастомных фильтров. Если бы у нас был еще один, то мы могли бы указать, чтобы он выполнился позже  
@Component // Filter должен быть бином  
// GlobalFilter - если хотим, чтобы фильтр применялся ко всему трафику, что приходит в gateway  
public class RequestTraceFilter implements GlobalFilter {  
    private static final Logger logger = LoggerFactory.getLogger(RequestTraceFilter.class);  
  
    @Autowired  
    FilterUtility filterUtility;  
  
    // Mono - один объект в reactive, Flux - коллекция объектов. Т.е. фильтр ничего не возвращает, то Mono<Void>  
    @Override  
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {  
        HttpHeaders requestHeaders = exchange.getRequest().getHeaders(); // получаем информацию о запросе  
        if (isCorrelationIdPresent(requestHeaders)) {  
            logger.debug("eazyBank-correlation-id found in RequestTraceFilter : {}",  
                    filterUtility.getCorrelationId(requestHeaders));  
        } else {  
            String correlationID = generateCorrelationId();  
            exchange = filterUtility.setCorrelationId(exchange, correlationID);  
            logger.debug("eazyBank-correlation-id generated in RequestTraceFilter : {}", correlationID);  
        }  
        return chain.filter(exchange); // chain of responsibility, т.е. даем другим фильтрам обработать запрос  
    }  
  
    private boolean isCorrelationIdPresent(HttpHeaders requestHeaders) {  
        if (filterUtility.getCorrelationId(requestHeaders) != null) {  
            return true;  
        } else {  
            return false;  
        }  
    }  
  
    private String generateCorrelationId() {  
        return java.util.UUID.randomUUID().toString();  
    }  
}
```

```java
@Component  
public class FilterUtility {  
  
    public static final String CORRELATION_ID = "eazybank-correlation-id";  
  
    public String getCorrelationId(HttpHeaders requestHeaders) {  
        if (requestHeaders.get(CORRELATION_ID) != null) { // проверяем, нет ли в запросе уже хедера с ID.  
            List<String> requestHeaderList = requestHeaders.get(CORRELATION_ID);  
            return requestHeaderList.stream().findFirst().get(); // возвращаем значение ID по этому хедеру.  
        } else {  
            return null;  
        }  
    }  
  
    public ServerWebExchange setRequestHeader(ServerWebExchange exchange, String name, String value) {  
        return exchange.mutate().request(exchange.getRequest().mutate().header(name, value).build()).build();  
    }  
  
    public ServerWebExchange setCorrelationId(ServerWebExchange exchange, String correlationId) {  
        return this.setRequestHeader(exchange, CORRELATION_ID, correlationId);  
    }  
  
}
```

Теперь напишем ResponseFilter. Разница с RequestFilter в том, что добавляется .then, в котором и пишется обработка. 
```java
@Configuration  
public class ResponseTraceFilter {  
  
    private static final Logger logger = LoggerFactory.getLogger(ResponseTraceFilter.class);  
  
    @Autowired  
    FilterUtility filterUtility;  
  
    // 2-ой способ создания фильтров. Создатить Bean типа GlobalFilter(хотя это не назвать вторым способом, просто  
    // в первом случае мы для бина создали отдельный класс, а тут не создавали)    @Bean  
    public GlobalFilter postGlobalFilter() {  
        return (exchange, chain) -> {  
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {  
                HttpHeaders requestHeaders = exchange.getRequest().getHeaders();  
                String correlationId = filterUtility.getCorrelationId(requestHeaders);  
                logger.debug("Updated the correlation id to the outbound headers: {}", correlationId);  
                exchange.getResponse().getHeaders().add(filterUtility.CORRELATION_ID, correlationId);  
            }));  
        };  
    }  
}
```

Чтобы включить вывод логов уровня DEBUG в пакете com.eazybytes.gatewayserver:
![[Pasted image 20241205152755.png]]

### Теперь в микросервисах, куда придет запрос из Gateway, в котором будет добавлен header мы хотим получить значение этого хедера и залогировать
Получаем значение хедера с помощью @RequestHeader(название хедера). Логируем и потом отправляем этот же хедер в запросы к другим микросервисам(если такие запросы есть)
![[Pasted image 20241205154221.png]]
Добавили в feignclient хедер, который нужно будет передать при запросе.
![[Pasted image 20241205154333.png]]

### Design Patterns around Api Gateway
#### Api Gateway
![[Pasted image 20241205161404.png]]
То, что существует единая точка входа для всех запросов.
#### Gateway Routing Pattern
![[Pasted image 20241205161536.png]]
То, что Gateway может выполнять routing запросов до микросервисов.
#### Gateway offloading Pattern
![[Pasted image 20241205161652.png]]
По сути говорит о том, что мы можем в нашем Gateway реализовывать различные cross-cutting concerns.
#### BFF
![[Pasted image 20241205161818.png]]
Для каждого вида клиентов создается свой отдельный Gateway.
Это может быть полезно, когда мы хотим в зависимости от клиента производить разные ответы. Для мобилок, т.к. часто они используют мобильный интернет, мы можем начать дополнительно сжимать данные на сервере, в то время как веб-клиент зачастую имеет стабильный интернет, поэтому сжатием данные можно пренебречь. 
#### Gateway Aggregator
![[Pasted image 20241205162312.png]]
Когда клиенту нужно получить информацию сразу с нескольких микросервисов, то можно в Gateway реализовать такой функционал, что он сам сделает запросы в несколько микросервисов, получит с них информацию и соберет агрегированный объект с информацией о клиенте. Если такой функционал есть в нашем Gateway, то это реализация паттерна Gateway Aggregator.
### Docker-compose 
Наследуемся от microservice-eureka-config из common-config.yml. В нем будет все для получения конфига с Configuration Server, а также для подключения к Eureka. Помимо этого, т.к. мы хотим, чтобы Gateway запускался самым последним, после того, как все микросервисы подняты, то мы добавляем ему depends_on: на все микросервисы, а для микросервисов прописывываем healthcheck, как мы это делали с configuration server.
![[Pasted image 20241205163916.png]]
![[Pasted image 20241205163928.png]]
