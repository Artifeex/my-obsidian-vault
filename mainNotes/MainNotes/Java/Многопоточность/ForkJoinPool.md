Это базовый threadPool, который создается Java приложением. Он используется, например, когда мы используем CompletableFuture как дефолтный пул для выполнения. Также используется в Stream.parallel.

 К тому же, мы сами можем передать туда задачки.

Есть статический метод в ForkJoinPool, который возвращает объект ForkJoinPool(т.е. паттерн одиночка, чтобы был всего один ForkJoinPool на все Java приложение)

ForkJoinPool.common() - вернет как раз объект ForkJoinPool, который реализует знакомый нам интерфейс ExecutorService. Т.е. так мы можем получить доступ именно к тому, который по дефолту создается в начале работы программы. При этом никто не мешает создавать через оператор new свои собственные ForkJoinPool.

Задачи хранятся
### Является work-strealing. Т.е. если какой-то поток выполнил свою задачу, то он может взять задачу из очереди другого потока.
В каком порядке хранятся задачи? Задачи добавляются в начало очереди. Т.е. тут получается как стек. И выполняются тоже сначала, т.е. получается, что сначала выполняются задачи, которые только пришли, а не те, которые долго стоят в очереди. Почему так? Потому что когда пришла новая задача, то есть шанс, что в кэше процессора осталась нужная для выполнения информация и тогда мы сможем быстрее выполнить задачу. К тому же, другие потоки будут брать задачи как раз с конца очереди! Т.е. мой поток выполняет задачи с вершины, а поток, который простаивает и решил украсть мою задачу выполняют задачи из конца очереди.


**Существующая связь между параллельными потоками и инфраструктурой ForkJoinPool заключается в базовой реализации параллельных потоков. Когда вы создаете параллельный поток, он использует ForkJoinPool по умолчанию, предоставленный Java, для параллельного выполнения операций потока. Это означает, что работа по разделению данных и их распределению по нескольким потокам выполняется ForkJoinPool.**

Т.е. важно, что распараллеливанием задачи занимается именно ForkJoinPool, а не Stream. Т.е. да, мы попросили stream выполнить задачу параллельно, но вот разбиением задачи занимается именно ForkJoinPool.