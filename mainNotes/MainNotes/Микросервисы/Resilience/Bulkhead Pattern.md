Bulkhead - это перемычка. Например, в короблях внутри делают комнаты, чтобы если произошла какая-то пробоина, то корабль не потонул сразу. Вода заполнит комнаду, а другие комнаты не будут заполнены из-за как раз разделения на такие комнаты.

В случае микросервисов мы также хотим иметь разделение внутри нашего микросервиса, чтобы, если какой-то endpoint имеет повышенную нагрузку, то он не начал забирать ресурсы у обработчика другого endpoint-а, тем самым "утопив" весь микросервис своей работой.
![[Pasted image 20241206170136.png]]
Spring Cloud Gateway такое не поддерживает. Но Resilience4j поддерживает.
Указываем над методом тип THREADPOOL и тогда для конкретного метода будет выделен тредпул, который он может использовать и он не сможет забирать другие ресурсы.
![[Pasted image 20241206170529.png]]
И мы можем настраивать максимальное количество параллельных запросов на какой-то метод, можем настраивать тред пул.
![[Pasted image 20241206170515.png]]