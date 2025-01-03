Контейнер сервлета управляет жизненным циклом сервлета, а также обеспечивает программную среду для выполнения самого сервлета. 
### Задачи Servlet контейнера

- Поддержка обмена данными. Контейнер сервлетов предоставляет легкий способ обмена данными между веб клиентом (браузером) и сервлетом. Благодаря контейнеру нет необходимости создавать слушателя сокета на сервере для отслеживания запросов от клиента, а так же разбирать запрос и генерировать ответ. Все эти важные и комплексные задачи решаются с помощью контейнера и разработчик может сосредоточиться на бизнес логике приложения.
    
- Управление жизненным циклом сервлетов и ресурсов. Начиная от загрузки сервлета в память, инициализации, внедрения методов и заканчивая уничтожением сервлета. Контейнер так же предоставляет дополнительные утилиты, например JNDI, для управления пулом ресурсов.
    
- Поддержка многопоточности. Контейнер самостоятельно создает новую нить для каждого запроса и предоставляет ей запрос и ответ для обработки. Таким образом сервлет не инициализируется заново для каждого запроса и тем самым сохраняет память и уменьшает время до обработки запроса.
    
- Поддержка JSP. JSP классы не похожи на стандартные классы джавы, но контейнер сервлетов преобразует каждую JSP в сервлет и далее управляется контейнером как обычным сервлетом.
    
- Различные задачи. Контейнер сервлетов управляет пулом ресурсов, памятью приложения, сборщиком мусора. Предоставляются возможности настройки безопасности и многое другое.

### Cookies в сервлетах

Cookies (куки) используются в клиент-серверном взаимодействии и они не являются чем-то конкретным к Java. Servlet API предоставляет поддержку cookies через класс javax.servlet.http.Cookie implements Serializable, Cloneable. Для получения массива cookies из запроса необходимо воспользоваться методом HttpServletRequest getCookies(). Для добавления cookies в запрос методов не предусмотрено.

Аналогично HttpServletResponse addCookie(Cookie c) — может добавить cookie в response header, но не существует геттера для этого типа передачи данных.**