![[Pasted image 20240925200959.png]]
![[Pasted image 20240925201029.png]]
Всегда есть source - т.е. источник данных. Далее идут промежуточные операции и в конце ВСЕГДА один терминальный оператор. Не может быть несколько терминальных операторов.

### Промежуточные операции
- filter - отфильтровать стрим по какому-то условию.
- map - преобразование одного объекта в другой объект.
- sorted - сортировка. Можно передать компаратор, по которому нужно сравнивать элементы в стриме.
- [[flatMap]] - используется для того, чтобы перевести Stream< Stream> в Stream объектов. 
- distinct - удаляет дубликаты из стрима. Остаются только уникальные элементы.
- skip - пропускает сколько-то первых элементов стрима.

### Терминальные операции
1. Используются для того, чтобы Stream собрать в коллекцию.
2. Есть операция count, которая возвращается количество элементов в стриме.
3. reduce - сворачивает все элементы, которые у нас есть в стриме в какой-то один объект.
4. anyMatch - принимает предикат, который проверяет, есть ли хотя бы 1 элемент в стриме, который удовлетворяет предикату. Возвращает true или false.
5. forEach - делает какую-то логику для каждого элемента.

### Особенности Stream 
Главной особенностью является то, что все стримы всегда ленивые. Когда мы вызываем промежуточные операции на самом деле под капотом ничего не происходит. Все операции накапливаются внутри и отрабатывают только тогда, когда вызывается терминальный оператор. Если терминальный оператор не вызовется, то никакие промежуточные операции не будут выполнены.

После того, как терминальный оператор отработал мы больше никак не можем использовать стрим.

Мы можем использовать лямбда выражения в методах стрима. Благодаря этому можно писать код в функциональном стиле.

Возможность использовать параллелизм. Можно добавить оператор parallel. Под капотом он сам все параллелит. Для параллелизма используется стандартный pool потоков CommonWorkJoinPool(количество потоков в нем равно количеству ядер в процессоре на нашей машине).

--------------------------------

![[Pasted image 20240926121908.png]]
Иллюстрация, которая показывает, что может происходит в стриме:
![[Pasted image 20240926122107.png]]
Входная труба - источник. Дальше синие штучки сжимают входные данные(map), потом пакман ест какие-то определенные квадратики(filter) и потом те, что прошли все эти этапы выводятся на экран(forEach).
Для увеличение перформанса можно использовать различные специализации.
![[Pasted image 20240926122340.png]]

### Источники Stream
![[Pasted image 20240926122440.png]]
- empty() - пустой стрим. Например, по интерфейсу должны вернуть стрим, но у нас нет элементов, поэтому возвращаем пустой.
- of(x, y, z) - явно передаем какие элементы хотим, чтобы были в стриме.
- ofNullable(x) - если x = null, то вернутся пустой стрим, а если не равен, то стрим из одного элемента.
- generate() - передается Supplier. В таком случае будет стрим из бесконечного числа элементов. 
- iterate - если хотим стрим, где следующее значение зависит от предыдущего. Начиная с Java 9 можем еще передать предикат, который ограничивает стрим. Если хотим какой-то range элементов в стриме, идущих друг за другом, то лучше использовать IntStream.range(1, 100) - стрим из 99 элементов от 1 до 99.
- collection.stream() - в любой коллекции есть метод .stream()
- Arrays.stream(array) - если хотим получить стрим из массива, то массив может быть только int, long, double, Object.
- Random.ints - стрим рандомных чисел.
- String.chars() - вернет стрим целых чисел(т.к. у нас нет CharStream, а есть IntStream, но надо будет просто помнить, что это просто коды символов)
- Можем писать свои источники

### Промежуточные операции
![[Pasted image 20240926123210.png]]
- map - отображает каждый элемент стрима с помощью какой-то функции. С помощью mapToInt и т.д. можно переключаться между различными специализациями. Не меняет количество элементов в стриме.
- filter - выкинуть из стрима часть элементов по предикату. Из интересного, filter не влияет на сортированность. Т.е. если стрим был отсортирован, то порядок элементов не изменится.
- [[flatMap]]
- mapMulti - разновидность flatMap(не видел реальных использований)
- distinct - избавиться от повторяющихся элементов в стриме.
- sorted - отсортировать стрим.
- limit - ограничивает число элементов в стриме. Если мы поставили лимит = 10, то сколько бы на источнике не было элементов останется максимум 10. Все остальные будут обрезаны. Так можно превратить бесконечный стрим в конечный.
- skip - пропустить сколько-то элементов. Чаще всего пропускают 1-ый элемент)
- peek - часто используется для отладки. Внутрь передается consumer, который принимает элемент стрима. Нужна для того, чтобы подглядеть(peek) за тем, какие же там элементы проходят в стриме.
- takeWhile - брать элементы, пока удовлетворяют условию. Как только условие не выполнилось, стрим будет обрезан и дальнейшие элементы из источника не будет добавлены в стрим.
- dropWhile - игнорировать элементы, пока они удовлетворяют указанному предикату.
- boxed - используется для примитивных стримов. Позволяет IntStream превратить в Stream< Integer>

Примеры:
Как рандомизировать порядок элементов в стриме? Вот так, как написано ниже делать не надо:
![[Pasted image 20240926124011.png]]
Проблема в том, что он работае для 10 элементов. Но если мы делаем 50 элементов, то вылетаем с ошибкой:
![[Pasted image 20240926124205.png]]
Это происходит потому что мы нарушаем контракт Comparatora. И для сортироку 9 элементов используется алгоритм, которые игнорирует то, что такой контракт нарушился, а уже после некоторого числа элементов используется другой алгоритм, который не игнорирует.

Правильный ответ в том, что мы не можем средствами StreamApi выполнить это. Если нам такое нужно, то собираем все в List и вызываем метод shuffle
![[Pasted image 20240926124433.png]]

Порядок операций в стриме важен:
Если мы вызываем sorted, то говорим, что хотим, чтобы отсортировались все элементы стрима(а он у нас бесконечный) и потом первые 10. Таким образом, получаем ошибку:
![[Pasted image 20240926124641.png]]
А вот здесь мы сначала limit(10), т.е. обрезали стрим, и потом этот обрезанный стрим отсортировали.
![[Pasted image 20240926124658.png]]

Мы можем смотреть, как работает стрим. Если в дебаге остановимся на строчке, в котором только стрим создался, то появится кнопочк, в которой можно удобно смотреть, как изменяется входящий стрим после каждого промежуточного метода.
![[Pasted image 20240926140813.png]]

### Терминальные операции и коллекторы
![[Pasted image 20240926140917.png]]
- count - количество элементов в стриме
- findFirst/findAny - найти первый/найти любой. В не параллельных стримах работают одинаково. В параллельных может дать преимущество findAny, поскольку когда стрим распараллелился, то если один из потоков что-то нашел, то выполнился findAny. А если мы искали findFirst, то уже параллельности не будет, т.к. только один из потоков будет работать с началом стрима даже в параллельности.
- anyMatch - вернет true если хотя бы эти элемент стрима удовлетворяет условию. 
- noneMatch - вернет true, если не один из элементов стрима не удовлетворяет условию
- allMatch - вернет true, если все элементы стрима удовлетворяют условию.
- forEach/forEachOrdered - выполнить какую-то операцию над всеми элементами стрима. 
- max/min - максимальный или минмальный элемент
- toArray - собрать стрим в массив(примитивный, либо объектный)
- toList - собрать стрим в неизменяемый список.
- sum/average/summaryStatistics - для специализаций.(summaryStatistics - хранит минимум, максимум, сумму и количество)

Примеры:
Тут peek не будет работать, потому что есть оптимизации. И вместо вызова count мы можем сразу вернуть значение 5, т.к. еще на этапе компиляции знаем, что всего 5 элементов в стриме.
![[Pasted image 20240926141814.png]]

![[Pasted image 20240926142334.png]]

Так можно проверить, есть ли хотя бы один элемент в стриме, удовлетворяющий условию:
![[Pasted image 20240926142646.png]]

Есть особенность при работе с anyMatch и findFirst. Если мы нашли решение, т.е. они выполнились, то нам же необязательно весь оставшийся стрим перебирать. И после этого происходит сигнал "остановись".
![[Pasted image 20240926142843.png]]

reduce() - когда хотим завершить стрим, но при этом хотим выполнить что-то свое, чего нет в методах, которые предлагаются. Например, можно посчитать факториал:
![[Pasted image 20240926143458.png]]
Есть 3 оверлоада:
1. Передаем первым параметром единичный объект. Т.е. объект, к которому можно применить функцию редукции и он не изменит ничего. В примере выше значение "1" не повлияет на вычисление факториала. Вторым параметром передается функция редукции. Она применяет 2 элемента стрима и возвращает 1 элемент. Таким образом, все придет к одному элементу.
2. Принимает только функцию для вычисления, которая из двух элементов стрима возвращает 1. Т.е. не используется какое-то начальное значение.
![[Pasted image 20240926144903.png]]
1. Плохо по производительности использовать reduce, поскольку для каждой суммы придется создавать свой объект Integer, что вызовет большую нагрузку на GC. Лучше использовать mapToInt и получить IntStream и у него уже вызвать sum() для получения суммы всех чисел стрима.
2. максимальный элемент стрима через редукцию искать глупо, когда есть метод max
3. у нас есть стрим строк, хотим собрать их в одну строку. Будет очень медленно из-за постоянно выделения новых объектов. Для этого пользуемся методом collect.

Если хотим получить из стрима список, то всегда пользуемся либо collect, либо toList, либо toArray.
![[Pasted image 20240926145706.png]]

### Collector
Передаются внутрь терминального метода collect. Базовые хранятся внутри Collectors
![[Pasted image 20240926145754.png]]

![[Pasted image 20240926145928.png]]
- toList - получаем в результате список. Под капотом мы не знаем, какая реализация List используется. (Но в текущей реализации возвращается ArrayList). Но лучше на это не рассчитывать и использовать toCollection, если хотим, чтообы использовалась определенная реализация.
- toSet - получаем в результате множество.
- toCollection(supplier). supplier - должен вернуть пустую коллекцию, которая потом заполняется элементами стрима. Напримеря, supplier = ArrayList::new
- toMap(keyMapper, valueMapper). keyMapper - должна элементы стрима превратить в ключи. valueMapper - элементы стримы превратить в значения. Если ключ повторился у 2 элементов стрима, то будет кидаться исключение, но можем передать функцию(merger), которая будет решать эту проблему. Эту функция возьмет 2 элемента, у которых совпали ключи и выполнит нашу логику, чтобы вернулся всего 1 элемент. Этот 1 элемент и будет значением по этому ключу.
- joining - для склейки стрима строк в одну строку. separator - для добавления сепаратора. prefix, suffix - для добавления префикса и суффикса.
- groupingBy - 
- partitioningBy - 
- reducing/counting/mapping/minBy/maxBy ... - аналогично либо промежуточным либо терминальным операциям. Но отдельно они никогда не используются, т.к. есть их альтернативы. Смысл писать .collect(Collectors.maxBy), если есть просто .max. Они обычно используются в связке вместе с groupingBy и partitioningBy.

Использование groupingBy(по умолчанию в конце используется Collectors.toList).
Задача - хотим сгруппировать список строк по размеру. Внутрь groupingBy передаем классификатор, результат выполнения которого и используется для группировки. Т.е. если несколько строк потока вернули одинаковое значение, то они группируются. Ключом является возвращаемое значение этого метода.
![[Pasted image 20240926152025.png]]
Но допустим, хотим, чтобы значение были не списки строк, а одна строка, которая получилась соединением строк, находящихся в одной группе. Тут мы уже комбинируем несколько Collectors(и в таких ситуациях и используются те коллекторы, которые мы обсуждали ранее по типу maxBy)
![[Pasted image 20240926152301.png]]

![[Pasted image 20240926152740.png]]

Сгруппировать департаменты по начальнику:
![[Pasted image 20240926152910.png]]
Задача та же, но хотим, чтобы был список названий отделов, а не список отделов. Группируем по chief. А вот уже для значений мы используем mapping, которая берет элементы нашего стрима(а это Department), берет у них название и потом собирает в List. 
![[Pasted image 20240926153121.png]]

flatMapping работает похоже со всеми другими flat. Мы берем Stream< Users> внутри каждого отдела. Т.е. получаем работников отдела и потом они все собираются в один стрим. А дальше, чтобы не было дубликатов мы из стрима юзеров собираем все в set.
![[Pasted image 20240926153649.png]]

Задача - получить минимальную цену книги по каждой из категорий. Может показаться, что нужно использовать groupingBy, но на самом деле там будет очень не тривиальный код. 
![[Pasted image 20240926154635.png]]

особенностью groupingBy является то, что никогда не может получиться пустой группы. Т.е. группа всегда создается, если такой ключ был.

partitioningBy - похож на groupingBy, но он просто в качестве классифкатора использует предикат. Т.е. у нас всегд 2 группы: true и false. И особенность в том, что одна из групп может оказаться пустой! Т.е. все элементы стрима могли попасть в true.

В стримах нет функции findLast, а бывает хочется найти именно последний элемент стрима. Для этого можно воспользоваться методом reduce. И возвращать из двух элементов каждый раз второй элемент.
![[Pasted image 20240926160014.png]]

### Как работает обработка элементов в `Stream`:

1. **Получение элемента из источника**:
    - `Stream` получает **один элемент** из источника данных (например, из коллекции, файла, генератора и т.д.).
2. **Применение промежуточных операций**:
    - Этот элемент проходит через **все промежуточные операции** (`map()`, `filter()`, и т.д.) в порядке их вызова.
    - Промежуточные операции, такие как `map()`, `filter()`, не выполняют вычисления немедленно, они лишь описывают цепочку преобразований.
3. **Выполнение терминальной операции**:
    - Когда все промежуточные операции для этого элемента завершены, результат передаётся в **терминальную операцию** (например, `collect()`, `forEach()`), которая завершает обработку этого элемента.
4. **Получение следующего элемента**:
    - После этого берётся **следующий элемент** из источника, и он проходит через ту же цепочку операций.

Процесс продолжается до тех пор, пока все элементы источника не будут обработаны.

При этом важно учитывать, что есть операции, такие как sorted или min/max, которые требуют сразу всех данных из источника, т.е. нужно их сохранить где-то. И тут может быть опасно, поскольку если файл большой, то может не хватить памяти, чтобы все сохранить.

`IntStream`, `LongStream` и `DoubleStream` в Java работают **с примитивными типами** под капотом, что позволяет избежать накладных расходов, связанных с использованием объектов-обёрток (`Integer`, `Long`, `Double`). Это сделано для повышения производительности, поскольку операции с примитивными типами быстрее и требуют меньше памяти.