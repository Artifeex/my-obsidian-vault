Concurrent Mark Sweep GC
Использование CMS GC включается опцией `-XX:+UseConcMarkSweepGC`

Мы уже встречали слова Mark и Sweep при рассмотрении последовательного и параллельного сборщиков (если вы не встречали, то сейчас как раз самое время это [сделать](https://habrahabr.ru/post/269707/)). Они обозначали два шага в процессе сборки мусора в старшем поколении: пометку выживших объектов и удаление мертвых объектов. Сборщик CMS получил свое название благодаря тому, что выполняет указанные шаги параллельно с работой основной программы.

При этом CMS GC использует ту же самую организацию памяти, что и уже рассмотренные Serial / Parallel GC: регионы Eden + Survivor 0 + Survivor 1 + Tenured и такие же принципы малой сборки мусора. **Отличия начинаются только тогда, когда дело доходит до полной сборки. В случае CMS ее называют _старшей (major) сборкой_, а не полной, так как она не затрагивает объекты младшего поколения.** В результате, малая и старшая сборки здесь всегда разделены. Одним из побочных эффектов такого разделения является то, что все объекты младшего поколения (даже потенциально мертвые) могут играть роль корней при определении статуса объектов в старшем поколении.  
  
Важным отличием сборщика CMS от рассмотренных ранее является также то, что он не дожидается заполнения Tenured для того, чтобы начать старшую сборку. Вместо этого он трудится в фоновом режиме постоянно, пытаясь поддерживать Tenured в компактном состоянии.  
  
Давайте рассмотрим, что из себя представляет старшая сборка мусора при использовании CMS GC.

Начинается она с остановки основных потоков приложения и пометки всех объектов, напрямую доступных из корней. После этого приложение возобновляет свою работу, а сборщик параллельно с ним производит поиск всех живых объектов, доступных по ссылкам из тех самых помеченных корневых объектов (эту часть он делает в одном или в нескольких потоках).

Естественно, за время такого поиска ситуация в куче может поменяться, и не вся информация, собранная во время поиска живых объектов, оказывается актуальной. Поэтому сборщик еще раз приостанавливает работу приложения и просматривает кучу для поиска живых объектов, ускользнувших от него за время первого прохода. При этом допускается, что в живые будут записаны объекты, которые на время окончания составления списка таковыми уже не являются. Эти объекты называются _плавающим мусором (floating garbage)_, они будут удалены в процессе следующей сборки.

После того, как живые объекты помечены, работа основных потоков приложения возобновляется, а сборщик производит очистку памяти от мертвых объектов в нескольких параллельных потоках. При этом следует иметь в виду, что после очистки не производится упаковка объектов в старшем поколении, так как делать это при работающем приложении весьма затруднительно.

![[Pasted image 20241113095923.png]]

Отдельно следует рассмотреть ситуацию, когда сборщик не успевает очистить Tenured до того момента, как память полностью заканчивается. В этом случае работа приложения останавливается, и вся сборка производится в последовательном режиме. Такая ситуация называется _сбоем конкурентного режима (concurrent mode failure)_. Сборщик сообщает нам об этих сбоях при включенных опциях `-verbose:gc` или `-Xloggc:filename`.  
  
У CMS есть один интересный режим работы, называемый Incremental Mode, или i-cms, который заставляет его временно останавливаться при выполнении работ параллельно с основным приложением, чтобы на короткие периоды высвобождать ресурсы процессора (что-то вроде АБС у автомобиля). Это может быть полезным на машинах с малым количеством ядер. Но данный режим уже помечен как не рекомендуемый к применению и может быть отключен в будущих релизах, поэтому подробно его разбирать не будем.

### Ситуации STW

  
Из всего сказанного выше следует, что при обычной сборке мусора у CMS GC существуют следующие ситуации, приводящие к STW:  

- Малая сборка мусора. Эта пауза ничем не отличается от аналогичной паузы в Parallel GC.
- Начальная фаза поиска живых объектов при старшей сборке (так называемая _initial mark pause_). Эта пауза обычно очень короткая.
- Фаза дополнения набора живых объектов при старшей сборке (известная также как _remark pause_). Она обычно длиннее начальной фазы поиска.

  
В случае же возникновения сбоя конкурентного режима пауза может затянуться на достаточно длительное время.