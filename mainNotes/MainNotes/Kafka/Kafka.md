Kafka - распределенный брокер сообщений в стриминговом режиме.
![[Pasted image 20241104181042.png]]
![[Pasted image 20241104181155.png]]
- распределенность - множество серверов, работающих слаженно.
- высокая доступность - выход из строя какого-то узла не нарушает доступ к данным.
- ACID(AC - обеспечивает kafka)
- горизонтальное масштабирование - можно добавлять множество не очень сильных компьютеров и все балансируется.

### Решаемая задача
Есть некоторые сообщения, которые хотим доставить. Есть производители этих сообщений и потребители. И может потребоваться доставлять какие-то типы сообщений одному какому-то потребителю, а другой тип сообщений нужно доставить сразу всем и т.д. Т.е. должна быть возможность широковещательной рассылки.
![[Pasted image 20241104181615.png]]
Какое решение? Добавить посредника, а не отправлять сообщения из producers напрямую в consumers. И этим посредником и является брокер сообщений.
Внутри него есть ящечки для определенного типа сообщений, в который кладутся сообщения от producers. А consumers ходят в broker и получают сообщения.
![[Pasted image 20241104181817.png]]

![[Pasted image 20241104181903.png]]
### Основные сущности kafka
![[Pasted image 20241104182004.png]]
### Broker
![[Pasted image 20241104182016.png]]
Если бы broker был один, то вывод его из строя = смерть всего обмена сообщениями. Поэтому несколько брокеров объединяются в кластер и общаются между собой.
![[Pasted image 20241104182110.png]]
Zookeeper - координатор, в котором хранится состояние и конфигурация кластера. zookeeper - небольшая БД, которая очень быстро работает с операциями чтения, но не очень хорошо с операциями записи. И вот для хранения метаинформации о состоянии Kafka кластера он и нужен.
![[Pasted image 20241104182205.png]]
Также среди брокеров определяется один Controller
![[Pasted image 20241104182413.png]]
### Сообщения
![[Pasted image 20241104182506.png]]
### Kafka Topic - это какое-то логическое объединение сообщений.
Т.е. у нас есть множество событий(сообщений) и мы их решили логически объединить в один какой-то topic и мы за producer кладем их в этот topic(причем могут приходить сообщениях из разных продюсеров). Представляет из себя очередь, т.е. сообщения кладутся в том порядке, в котором их отправляли. Извлекаются потребителями также в таком же порядке.
![[Pasted image 20241104182712.png]]
Причем важно заметить, что данные не удаляются из очереди при чтении. Может прийти другой потребитель и также прочитать данные из этого же топика. Благодаря этому и обеспечивается широковещательные сообщения.
### Но это мы рассмотрели все в одном потоке по сути. А когда работаем с big data и большой нагрузкой, то явно нужна многопоточность. Для этого были придуманы partitions(партиции)
По сути каждая партиция - это отдельная очередь(их количество можно задавать). Сообщения внутри партиций распределяются каким-то образом, но не по порядку. Порядок есть только при чтении из какой-то определенной партиции. Т.е. за потребителя мы можем считывать внутри партиции только в том порядке сообщения, в котором они приходили. И так, например, поступают с действиями пользьзователя. К какой-то партиции можно привязать пользователя и его действия и потом за потребителя считывать из этой партиции ивенты в том порядке, в котором их совершал пользователь и производили производители. ![[Pasted image 20241104183221.png]]
Партиции распределены по брокерам. Нумируюстя с 0 и т.д. 
![[Pasted image 20241104183831.png]]
![[Pasted image 20241104183916.png]]
![[Pasted image 20241104183932.png]]
Почему так? Потому что кафра распределят равна партиции между брокерами и неважно, к какому топику принадлежат партиции. Т.е. например, у нас 450 партиций и 3 брокера. Тогда по 150 партиций получает каждый брокер и может получиться так, что все партиции какого-то топика могут лежат в одном брокере.

Но такое распределеннее может быть плохим, поскольку какой-то топик и его партиции могут быть очень загруженными, т.е. по ним постоянно гоняются сообщения, и, следовательно, повышенная нагрузка, а на других брокерах с другими партициями - наоборот, мало данных. Тогда можно вручную заняться балансировкой партиций по брокерам, используя конфигурационный файл. 
### Где хранятся данные, которые мы отправили в топик?
Они хранятся в log файлах.
![[Pasted image 20241104184347.png]]
Есть папка ./logs и внутри ее еще папки <название топика>/<номер партиции>
![[Pasted image 20241104184502.png]]
И внутри уже какой-то конкретной папки мы увидим 3 файла:
![[Pasted image 20241104184553.png]]
Внутри .log хранится само сообщение. 
- offset - номер сообщения в партиции.
- position - это смещение в этом файле в байтах.
![[Pasted image 20241104184612.png]]
.index - это маппинг оффсета на позицию. Т.е. хотим прочитать по offset = 1. Идем в этот файл, узнаем, сколько надо сделать смещений в байтах. После этого открываем файл .log и смещаемся относительно начала на 67 байт(если смотреть на пример на картинке ниже).
![[Pasted image 20241104184717.png]]
.timeindex - маппинг timestamp на offset. Например, мы хотим прочитать сообщения от вторника, а мы не знаем с какого offset считать. И вот как раз идем в этот файлик, указываем timestamp и узнаем offset с которого считать.
![[Pasted image 20241104184827.png]]
Если наш топик поработает подольше, то увидим, что появятся еще файлики с таким же расширением, но имя изменилось.
![[Pasted image 20241104185043.png]]
У log файл есть лимит(1 гигабайт по умолчанию) и когда он достигается, то создается новый, а старый закрывается. Это называется сегментами. В kafka всегда активен только 1 сегмент, старый морозится.
### Как удалить данные из топика?
![[Pasted image 20241104185342.png]]
Но это порождает проблему - если в сегмент сохранится сообщение, с каким-то далеко в будущем timestamp-ом. Например, кто-то по какой-то причине установил для своего сообщения timestamp через год. Тогда этот сегмент, в который попало сообщение будет жить очень долго.

### Репликация данных - надежность и отказоустойчивость
Если бы все данные какой-то одной партиции хранились в одном экземпляре на одном брокере, то выход из строя этого брокера автоматически терял бы данные партиции.
![[Pasted image 20241104185646.png]]
Есть такая настройка - replication-factor - число копий партиции. Например, если он = 2, то создается две копии каждой партиции и важно, что каждая копия партиции работает на разных брокерах, чтобы не было такого, что да, у нас 2 копии, но при этом они находятся на одном брокере и падение брокера = потеря данных партици. П
![[Pasted image 20241104185740.png]]
Причем если останется всего одна партиция(если ее копия оказалась на брокере, который вышел из строя), то создается еще одна копия.
![[Pasted image 20241104185936.png]]
![[Pasted image 20241104190032.png]]
![[Pasted image 20241104190051.png]]
Для обеспечения согласованности между репликами выбирается среди копий партиции - одна главная реплика(leader).
![[Pasted image 20241104190204.png]]
И есть важный нюанс - чтения и запись происходит только с leder репликой, остальные follower отдыхают.
![[Pasted image 20241104190211.png]]

![[Pasted image 20241104190246.png]]
Можем вручную перебалансировать, где находятся leader реплики.

### Как происходит синхронизация между leader и followers, которые должны в случае чего хранить актуальные данные, чтобы они не потерялись в случае отказа ноды, на которой работает leader 
Followers периодически опрашивают leader не появилось ли у него что-нибудь нового. 
![[Pasted image 20241104190508.png]]
Т.к. опрашивают периодически, а период хоть и маленький, но он есть, в который как раз может произойти какое-то изменение и потом leader упал. Как теперь среди follower выбрать нового лидера и узнать, все ли у него данные от прошлого лидера? И получается в таком варианте, нет никаких гарантий.
![[Pasted image 20241104190629.png]]
Т.к. прошлый вариант не давал гарантий, то ввели in-sync реплики.
![[Pasted image 20241104190910.png]]
В чем особенность таких реплик? Когда мы пишем событие в leader, то синхронно записываются данные и в isr реплики, т.е. получается синхронная запись. А оставшиеся фолловеры, которые не являются isr могут отставать. И получается, что при выходе из строя лидера следующего лидера мы можем выбрать из ISR фолловеров. Но ISR реплики замедляют систему, т.к. теперь нужно записать сообщение не только в leader, но и в ISR. 
min.insync.replicas = ставят обычно на 1 меньше, чем 4(или еще меньше). Почему так? Потому что есть такое требование, что если у нас нет столько ISR, сколько прописано в параметре min.insync.replicas, то сообщение просто не записывается(leader также входит в это количество, т.е. на скрине выше min.insync.replicas=3 - это leader + 2 ISR). И вот у нас получается 4 реплики и 3 из них ISR. И в случае падения какого-то нода с брокером мы сможем обеспечить min.insync.replicas=3, выбрав, например, обычного follower, в качестве нового ISR.

### Producer
![[Pasted image 20241104191446.png]]
![[Pasted image 20241104191532.png]]
У функции send есть гарантии доставки:
acks может принимать значения:
- 0 - producer не требует подтверждения об отправки сообщения
  ![[Pasted image 20241104191611.png]]
- 1 - возможна проблема, если после того, как leader принял сообщение и начал скидывать сообщения ISR репликам и упал, т.е. не успел скинуть данные, которые принял. Таким образом, сообщение потерялось, т.к. его не успели перекинуть в реплики.
  ![[Pasted image 20241104191708.png]]
- all - самый надежный режим. Получаем сообщение подтверждения доставки только после того, как leader записал сообщения во все ISR реплики.
  ![[Pasted image 20241104191843.png]]

А также у send есть семантика доставки:
- at most once - отправится не более одного сообщения
- at least once - хотя бы одно сообщение отправится и больше(значит, может отправить сообщение дублем)
- exactly one - строго одно сообщение отправится.
### Разберем подробнее этап отправки сообщения
![[Pasted image 20241104192038.png]]
Сначала producer должен получить метаданные о том, где находится leader в каком брокере и куда собственно отправлять сообщение + получить информацию о топиках и том, как что расположена в брокерах. Это все хранится в zookeper. В новых версиях kafka сама ходит в zookeper и отправляет метаинформацию, а раньше producer сам ходил в zookeeper.

И есть важный нюанс, что хоть операция send заявлена как асинхронная, т.е. мы выполнили отправку и пошли заниматься своими делами, а потом когда-то асинхронно придет подтверждение, что данные пришли. Но вот получение метаданных из zookeeper - это блокирующая операция, т.е. мы должны подождать, пока она выполнится, чтобы как раз узнать, куда сообщение то отправить. И если вдруг с zookeeper что-то не так, то все может заблокироваться на 60 сек(это timeout, который настраивается)
Поэтому ну нужно создавать много продюсеров. Нужно создать одного, который сходит в zookeeper, получит всю метаинформацию, закэширует ее у себя и потом будет ее использовать(и периодически обновлять кэши).

После получения метаданных мы полученное сообщение сериализуем в нужный формат.
![[Pasted image 20241104192808.png]]
Мы указываем key.serializer и value.serializer. Например, самый просто StringSerializer, который строки переводит просто в байты.

Следующим действие выбираем партицию, в которую пойдет сообщение.
![[Pasted image 20241104192902.png]]
Как можем указать?
- explicit partition - можем указать конкретный номер партиции.
- round-robin - без разницы в какую партицию, разберись сам. По умолчанию будем писать по очереди - сначал в первую партицию, потом во вторую и т.д.
- key-defined - партиция определяется по ключу(когда мы отправляем сообщение, то мы можем указывать ключ, который используется для распределения по кластеру. Гарантируется, что если мы отправляем сообщения с одним и тем же ключом, то они будут попадать в одну и ту же партицию. Технически - берется хэш ключа и остаток по модулю на количество партиций). Используется очень часто!

Дальше сообщение сжимается. Потом мы аккумулируем batch для повышения производительности. Т.е. мы не сразу отправляет сообщение, когда узнали какому брокеру отправить и в какую партицию, а собираем сообщения в batch. Причем в один batch кладутся сообщения направленные в определенную партицию, а не так, что в одном batch сразу несколько сообщений в разные партиции. И у batch есть две настройки:
- batch.size - после преодоления некоторого размера в байтах, батч отправляется. Т.е. сообщения кладутся в него и после того, как размер был превышен батчем отправляются в брокер.
- linger.ms - на случай, если сообщения нужно отправить, но они уже долго не могут превысить batch.size. Тогда ставится время и если батч не отправляется за указанное время по batch.size условию, то мы отправляем его по условию linger.ms( истечения времени).
![[Pasted image 20241104193410.png]]
Также есть нюанс, что если у нас в брокере с номером 2 в него копятся два batch. Один, например, в партицию с номер 1, а второй batch, в партицию с номером 2. И если они в сумме превышают batch.size, то мы сразу 2 batch и отправляем в этот брокер.

### Consumer
![[Pasted image 20241104194222.png]]
Есть команда poll, которой мы читаем сообщения пачками. Т.е. не по одному сообщению забираем, а забираем из партиции сразу пачку сообщений.

Сначала извлекаем метаданные. Где находятся топики, какие реплики лидеры, на каких брокерах они лежат, в каком состоянии кластеры.
![[Pasted image 20241104194533.png]]
Если у нас 1 consumer, то читает данные со всех партиций нужного топика, но это может быть довольно медленно, а лучше делать в многопоточке.
![[Pasted image 20241104194649.png]]
Kafka поддерживает многопоточку. Мы можем создать несколько consumers и указать у них одинаковый group.id. И теперь каждый consumer будет читать определенные партиции.  И таким образом, мы читаем в параллель. И вот в данном случае каждый consumer получил по партиции. Если мы добавим еще один, то он будет просто проставить, а если убавим на 1, то один из потребителей будет читать сразу с нескольких партиций. Поэтому в нагруженных приложениях нужно держать баланс.
![[Pasted image 20241104194658.png]]
### Offset у consumer
Допустим есть consumer, который читает из партиции 0. Брокер смог сразу одним сообщением доставить 3 сообщения из партиции.
![[Pasted image 20241104195024.png]]
После этого чтения consumer упал. Тогда другой consumer возьмет на себя работу с партицией 0. Но не хотелось бы новой читать сообщения 0,1,2.
![[Pasted image 20241104195015.png]]
Для решения проблемы есть отдельный topic - consumer_offsets. В нем хранятся offset для каждой группы, для каждой партиции. Т.е. хранится:
- partition - топик и партиция
- group - какая группа consumer(group.id) читает партицию.
- offset - сколько удалось прочитать. offset коммитит как раз тот consumer, который прочитал сообщение.
![[Pasted image 20241104195345.png]]

И когда происходит переключение чтения партиции 0, на другого consumer из группы, то сначала идет в топик consumer_offsets. Ищет информацию по offset. И потом уже читает из партици 0 сообщения по offset, т.е. пропускает 0,1,2 сообщения, которые прочитал consumer, который вышел из строя.
![[Pasted image 20241104195605.png]]
![[Pasted image 20241104195618.png]]

### Какие типы commit-ов есть?
![[Pasted image 20241104195809.png]]
- auto commit - не всегда хорош. Когда получили batch данных от брокера, то происходит сразу автокоммит и кафка считает, что для этой группы консьюемеров эта партиция сообщений обработана. Но по сути, могли произойти то, что потребитель получил сообщения, потом отправил коммит о получении и упал, т.е. сообщения никак не были обработаны, а kafka считает, что были и другой consumer уже не получит эти сообщения из-за offset и не обработает их. Такая семантика доставки называется at most once, т.е. мы можем как обработать, так и не обработать.
- manual commit - ручной коммит. Отправляет коммит только после того, как обработали сообщения. В данном случае происходит семантика at least once. Т.е. хотя бы раз сообщение будет обработано точно, поскольку только после обработки мы отправляет commit, но при этом сообщение может быть обработано и большее количество раз. Например, нам пришло 3 сообщения, мы 2 из них обработали, а 3-е не смогли и свалились, коммита тогда никакого не происходит и следующий consumer снова будет обрабатывать эти 3 сообщения и первых 2 из них будут обработаны дважды. И таким образом, нужно тратить дополнительные силы на то, чтобы проверить, а не обработали ли мы эти сообщения уже в другом consumere, который просто упал раньше, чем сделал commit.
- Но  есть еще exactly once семантика. Это когда мы и не хотим, чтобы сообщение потерялось, но также не хотим, чтобы оно могло обработаться дважды. Такая семантика является самой требовательной и сильнее всего тормозит систему.
	![[Pasted image 20241104200430.png]]
	Для обеспечения мы не полагаемся на topic из Kafka, а где-то у себя храним commit, в БД, в файле и т.д.
	Т.е. когда мы потребителем обращаемся к партиции, то передаем offset, с которого хотим читать сообщения. Сам же offset мы сохраняем у себя в БД после обработки. Т.е. мы никак не рассчитываем на тот offset, который хранит для нас kafka.

### Не активность Consumer Group при работе с какой-то партицией 
![[Pasted image 20241104200817.png]]
Если какая-то Consumer Group не работает с topic  "offsets.retention.minutes" времени(по умолчанию 7 дней), то в topc consumer_offsets она не найдет информацию об offset и тогда непонятно, откуда считывать сообщения, когда Consumer Group снова активизируется и начнет читать сообещния по топику. В таком случае есть 2 стратегии:
- earliest - считываем сообщения с самого начала 
- latest - старые сообщения не читаем, читаем новые, которые будут приходить в topic.
### Kafka performance
![[Pasted image 20241104201259.png]]
- Scalable architecture - горизонтальное масштабирование.
- Sequential write and read - нет рандомного чтения с переводом головки жесткого диска. Т.е. мы считываем последовательно данные из файлов(и записываем тоже) и поэтому головка жесткого диска физически не так сильно перемещается, как в случае random read.
- Zero-copy - это когда данные сообщения из файла могут быть переданы сразу же в socket приложения без промежуточной передачи сообщения в основное приложение. Это подходит, когда сообщение никак не преобразуется дополнительно приложением - а это как раз случай для kafka, т.к. consumers потребляет сообщения в том виде, в котором его передали producers.
- Множество настроек по различным компонентам и можно настроить под какие-то определенные кейсы.
![[Pasted image 20241104201759.png]]

### Гарантии доставки в Kafka
- [[At most once]]
- [[At least once]]
- [[Exactly once]]