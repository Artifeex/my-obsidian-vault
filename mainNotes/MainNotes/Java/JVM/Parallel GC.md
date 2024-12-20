Parallel GC (параллельный сборщик) развивает идеи, заложенные последовательным сборщиком, добавляя в них параллелизм и немного интеллекта. Если на вашем компьютере больше одного процессорного ядра и вы явно не указали, какой сборщик хотите использовать в своей программе, то почти наверняка JVM остановит свой выбор на Parallel GC. Он достаточно простой, но в то же время достаточно функциональный, чтобы удовлетворить потребности большинства приложений.

Для определения количества потоков, которые будут использоваться при сборке мусора, на компьютере с N ядрами процессора, JVM по умолчанию применяет следующую формулу: если N ≤ 8, то количество потоков равно N, иначе для получения количества потоков N домножается на коэффициент, зависящий от других параметров, обычно это 5/8, но на некоторых платформах коэффициент может быть меньше.  
  
**По умолчанию и малая и полная сборка задействуют многопоточность. Малая пользуется ею при переносе объектов в старшее поколение, а полная — при уплотнении данных в старшем поколении.  
  
Каждый поток сборщика получает свой участок памяти в регионе Old Gen, так называемый _буфер повышения (promotion buffer)_, куда только он может переносить данные, чтобы не мешать другим потокам. Такой подход ускоряет сборку мусора, но имеет и небольшое негативное последствие в виде возможной фрагментации памяти:
![[Pasted image 20241022093950.png]]
Интеллектуальная составляющая улучшений параллельного сборщика относительно последовательного заключается в том, что у него есть настройки, ориентированные на достижение необходимой вам эффективности сборки мусора. Вы можете указать устраивающие вас параметры производительности — максимальное время сборки и/или пропускную способность — и сборщик будет изо всех сил стараться не превышать заданные пороги. Для этого он будет использовать статистику уже прошедших сборок мусора и исходя из нее планировать параметры дальнейших сборок: варьировать размеры поколений, менять пропорции регионов.  
  
Например, если при малой сборке JVM не удается укладываться в отведенное вами время, размер младшего поколения может быть уменьшен. Если не удается достигнуть заданной пропускной способности, а с задержкой проблем нет, то размер поколения будет увеличен. И так далее.

### Ситуации STW
Как и в случае с последовательным сборщиком, на время операций по очистке памяти все основные потоки приложения останавливаются. Разница только в том, что пауза, как правило, короче за счет выполнения части работ в параллельном режиме.

### Достоинства и недостатки
Бесспорным плюсом данного сборщика на фоне Serial GC является возможность автоматической подстройки под требуемые параметры производительности и меньшие паузы на время cборок. При наличии нескольких процессорных ядер выигрыш в скорости будет практически во всех приложениях.  
  
Определенная фрагментация памяти, конечно, является минусом, но вряд ли она будет существенной для большинства приложений, так как сборщиком используется относительно небольшое количество потоков.  
  
В целом, Parallel GC — это простой, понятный и эффективный сборщик, подходящий для большинства приложений. У него нет скрытых накладных расходов, мы всегда можем поменять его настройки и ясно увидеть результат этих изменений.  
  
Но бывает так, что его оказывается недостаточно и нужно искать что-то более изощренное. О более продвинутых реализациях сборщиков мы поговорим уже в следующей статье.