Aspect-Oriented Programming
![[Pasted image 20241031085759.png]]
Делаем такой простой метод.
Например, хотим добавить дополнительный функционал с логированием:
![[Pasted image 20241031085825.png]]
Потом хотим добавить обработку исключений, поскольку уровень репозитория может выкидывать исключения, связанные с работой с БД
![[Pasted image 20241031085912.png]]
Далее мы добавляем security, т.к. не каждый пользователь может получить данные по id другого юзера. 
![[Pasted image 20241031085950.png]]
Потом еще хотим добавить работу с транзакциями. В общем, мы видим, что код все увеличивается и увеличивается. А нас будут интересовать только 2 строчки кода, где реально происходит findById. Поэтому хотелось бы вынести весь тот дополнительный код куда-то в другое место. И тут приходит на помощью CROSSCUTTING CONCERNS:
![[Pasted image 20241031090104.png]]
У нас есть потоки выполнения в нашей программе. И в процессе своей работы они могут проходить как раз через слои logging, transaction, если им потребуется. И в точках пересечения как раз и будет добавляться такой функционал. С этим нам поможет AOP.

### Терминалогия АОП
![[Pasted image 20241031090319.png]]
### Aspect
Аспект - это аналог класса. Именно в нем мы будем прописывать все остальные составляющие.
### Join Point
Это точка соединения в нашей программе, в которой мы и хотим добавить сквозной функционал(например, логирвоание или открытие транзакции)
### Advice
Совет. Это и есть тот сквозной функционал, который мы хотим добавить.
### Pointcut
По сути это предикат. С помощью него мы будем решать, нужно ли добавить advice в join point или нет. 
![[Pasted image 20241031090605.png]]

### Есть два основных подхода к AOP
![[Pasted image 20241031090714.png]]
AspectJ - библиотечка, которая работате на этапе компиляции. И раз она работает на этапе компиляции, то сквозной функционал(advice) можно вносить куда угодно. А в случае SpringAOP - он добавляем функционал с помощью proxy, поэтому можем добавить функционал только в методы. Но почти всегда хватает SpringAOP.

### Pointcut
Чтобы класс стал аспектом нужно добавить аннотацию @Aspect
Добавлять функционал мы можем только в бины.

Внутри интерфейса Pointcut есть два метода.
- Проверяет класс и возвращает true или false. Т.е. мы решаем, хотим ли добавить Join Point для этого класса или нет.
- А второй уже проверят конкретный метод класса. Т.е. мы же можем захотеть добавить функционал в какой-то конкретный метод класса, а не во все методы.

Внутри @Poincut можно писать всякие ключевые слова и выражения.
![[Pasted image 20241031091715.png]]
- @within() - проверяем ту или иную аннотацию над классом.
  ![[Pasted image 20241031091814.png]]
  Вот так, например, можно проверить, что класс является контроллером. И в целом все записи, которые начинаются с @ относятся к аннотациям.
- within - здесь мы уже указываем класс, а не аннотацию над классом. Например, вот так можем неаписать Pointcut, который вернет true(ну как бы классы пройдут проверку) для всех наших сервисов
  ![[Pasted image 20241031092059.png]]
- this - обращается к AOP proxy(который будет создан) и проверять его тип. target - к исходному объекту, вокруг которого будет обернут proxy и проверять его тип.
   ![[Pasted image 20241031092304.png]]
- @annotation - проверяет аннотацию над методом. Но тогда такая аннотация будет проверять вообще все бины и смотреть, нет ли у них над методами аннотации GetMapping. Но мы бы хотели, чтобы такая аннотация проверялась только у контроллеров, поэтому мы можем в одном Pointcut объединить сразу несколько. Вызываем isControllerLayer - тот pointcut, который мы написали ранее и добавляем логическое "И" уже проверка GetMapping. (Можно использовать логическои и, или, отрицание).
  ![[Pasted image 20241031092820.png]]
- args() - Можем проверять параметры наших методов. Например, args(Model, ..) - означает, что мы ожидаем, что в методе первым параметром будет идти объект типа Model, а дальше от 0 до бесконечности параметров любых типов. 
  args(Model, \*) - звездочка означает ровно 1 параметр. Т.е. для прохождения предиката нужно, чтобы в методе первым параметром шел Model, а дальше строго 1 параметр любого типа.
  ![[Pasted image 20241031093305.png]]
- @args - для проверки аннотаций, которые стоят над классом параметра. Т.е. у нас, например, есть Hibernate Entity. Мы сверху вешаем аннотцию @Entity и вот мы можем как раз проверить, что переданный параметр в метод имеет такую аннотацию.
  ![[Pasted image 20241031093625.png]]
- bean() - смотрит название бина. Ищем bean, имя которого заканчивается на Service.
  ![[Pasted image 20241031093743.png]]
- execution - следует определенному шаблону. 
  Шаблон:
  execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
  
  modifiers-pattern? - это модификаторы доступа, причем знак вопроса говорит, что это optional. Т.е. можем и не писать. Например, можем искать только public методы.
  Правила хорошего тона говорит, что мы в AOP ищем только public методы.
  ret-type-pattern - возвращаемый тип. И видим, что знака вопроса нет. Значит, он обязательный. Но если нам все равно, что там за возвращаемый тип, то можем поставить знак \*
  declaring-type-pattern? - указывает класс, в котором находится метод. Сразу за ним идет name-pattern. И чтобы разделить declaring-type-pattern и name-pattern между ними ставится точка. Т.е. будет declaring-type-pattern.name-pattern
  name-pattern - название метода.
  (param-pattern) - параметры(можем пользоваться * и .. если все равно на типы и количество параметров)
  throws-pattern? - какой exception ожидаем
  
  Найдем все public методы, возвращаемый тип не интересен, в наших классах Service, с именем findById и одним параметром любого типа.![[Pasted image 20241031094604.png]]


### Написание advice
Таким образом, мы написали Pointcut, куда и будет вставлена функциональность. Теперь напишем саму функциональность(advice).
@Before - для внесения функциональности перед выполнением какого-то метода.
Внесем функциональность логирования для findById методов сервисов. Для этого внутри Before ссылаемся на pointcut, который как раз решит, вставлять или не вставлять тот функционал, который мы напишем внутри уже самого метода, помеченного @Before
![[Pasted image 20241031095034.png]] Если поставим точку останова, то увидим, что вокруг нашего класса userService для которого мы написали advice с логированием создалось cglib прокси, с которым мы и работаем. ![[Pasted image 20241031095211.png]]
Каждый advice, который мы создаем - в aop proxy создается просто interceptor. И на скрине ниже мы видим, что последним идет как раз наш BeforeAdviceInterceptor, который мы написали. Где это все хранится? Это хранится как раз внутри aop прокси, которая создется поверх нашего объекта(в ней, кстати, в поле target хранится наш исходный объект.)
![[Pasted image 20241031095502.png]]
 Но мы можем заметить, что помимо нашего interceptora есть еще 3 интерсептора. Один из них TransactionInterceptor - который как раз занимается открытием и закрытием транзакции на уровне сервисов.
 Security - также работает с помощью AOP. Проверяет имеем ли мы права или нет на вызов каких-то методов.
 
 Схематично, что происходит:
![[Pasted image 20241031100540.png]]
Вызывается метод findById на AOP proxy, а не реального класса. Т.к. вызывается proxy метод, то у него есть списов advice, который располагаются в определенном порядке и вызываются сначала advice для этого метода. И только после всего этого вызывается метод реального объекта UserService. Таким образом, и работает Spring.

По умолчанию всегда создается cglib прокси для AOP. Только если в applaction.properties мы укажем spring.aop.proxt-target-class=false, только тогда будет использоваться dynamic proxy.

### Поговорим о том, как получить больше информации о том, куда мы внедрили advice
Для этого мы воспользуемся JoinPoint, которая знает, куда она внедрилась. 
Попросим просто внедрить JoinPoint в метод advice.
![[Pasted image 20241031101154.png]]
И в этом интерфейсе мы с помощью методов можем получать разную информацию о классе или методе, в который внедрились:
![[Pasted image 20241031101256.png]]
В advice мы можем не только ссылаться на готовые Pointcut, но и сами использовать те же ключевые слова, что и в Pointcut. А также, за счет этого получать их значения. Например, добавим args(id) - это означает, что этот advice сработает для метода, у которого есть 1 параметр с именем id. А в методах уже самого advice метода мы пишем такое же название id. И тогда в id передается значение из реального метода!
![[Pasted image 20241031101531.png]]
Таким образом, мы можем иметь доступ к значением параметров, классов и всего, что нам может потребоваться внутри advice методов.
 ![[Pasted image 20241031102121.png]]
 
 До java 8 нельзя было сохранять название аргументов, поэтому есть argNames, куда мы и передаем названия аргументов в наши методы ![[Pasted image 20241031102332.png]]
JoinPoint - всегда должен идти первым параметров, если он нам нужен. Если не нужен, то просто не пишем, а сразу используем значения из Pointcut.

### @After
Изучим @After advic-ы. Т.е. их несколько.
Между ними можно провести аналогию с try catch finally.
@Before - вызывается перед findById.
@AfterReturning - вызывается после работы findById, но только в том случае, если внутри него не произошло никаких исключений. Если исключение произошло, то вызывается уже @AfterThrowing, а @AfterReturning - нет. Если же наооброт, исключений не произошло, то @AfterThrowing - не вызывается. @After - как и блок finally вызывается всегда:
![[Pasted image 20241031102800.png]]

Стоит заметить, что мы можем получить доступ к возвращаемому значению только с помощью @AfterReturning, указав returning="название, по которому будем получать в параметрах метода". Другие ключевые слова из Poincut не помогут получить возвращаемое значение.
![[Pasted image 20241031103218.png]]

У @AfterThrowing есть доступ к exception, который произошел. 
![[Pasted image 20241031103415.png]]

### AOP Around 
Является самым сложным в том плане, что он включает в себя сразу несколько других Advice. Например, к нему относится Transactional. Нам нужно отловить момент перед вызовом метода, чтобы открыть транзакцию, потом отловить момент успешного завершения метода, чтобы вызвать commit транзакции и в случае какой-то ошибки нужно отловить эту ошибку и сделать rollback.
Напишем свой around. Для этого нам нужно получить доступ к вызову основного метода, а для этого нужно использовать уже ProceedingJoinPoint, в котором есть метод proceed, который и вызывает реальны метод и возвращает результат. Из нашего @Around advice нужно также вернуть результат вызова реального метода, а также выбросить исключение, если метод выбросил ислючение! И вот тут как раз мы и можем написать логику всех других advic-ов. Перед блоком try - то, что в @Before, в блоке try после вызова метода то, что @AfterReturning, то что в catch - @AfterThrowing, то что в finally - @After
![[Pasted image 20241031104848.png]]

### AOP-best-practices
Те pointcut, которые мы разбирали на самом деле можно разделить на 3 группы.
1-ая группа - уровня классов. Работают быстрее всех
- within
- @within
2-ая группа - относится к методам, т.е. то, что мы хотим дополнительно внедрить в наши методы advice дополнительно. (работают медленнее всего, а, значит, их нужно использовать вместе с другими pointcut-ами, которые будут быстрее работать, например, те, что на уровне классов):
- this
- args
- target
3-ая группа - execution. Те, которые по сути определяются на уровне методов.

Поэтому хороший pointcut должен содержать в себе 1-ый уровень, 3-ий уровень и по желанию уже добавляем 2 лвл, если нам нужно получить какую-то доп информацию. Тогда AOP будет работать быстро. А раз так, то у нас явно будут какие-то общие poincut, которые, например, определяют уровень сервисов, репозиториев и т.д. 
И тут мы приходим ко второму best-practice.

#### Выносить общие Pointcuts в отдельный класс
Выносим в отдельный класс.
![[Pasted image 20241031105840.png]]
И уже в других Aspect-ах мы просто указываем полный путь до Pointcut: ![[Pasted image 20241031105912.png]]

#### Мы должны использовать тот тип advice, который решит нашу проблему, а не писать @Around в том месте, где, например, нам бы хватило @After.

#### Нужно разделять по бизнес логике наши аспекты, а не писать все в одном. Т.е. в одном аспекте мы пишем логику по логированию, если захотели где-то поработать с транзакциями - то новый аспект, если с кэшем - еще один и т.д.

#### Мы можем управлять порядком выполнения аспектов. 
![[Pasted image 20241031110339.png]]
 ![[Pasted image 20241031110353.png]]
 Теперь FirstAspect - будет вызываться первым, а после него SecondAspect.
 SecondAspect(с AROUND, который внутри выполнял функции @Before)  - вызвался после advice c @Before внутри FirstAspect
 ![[Pasted image 20241031110433.png]]