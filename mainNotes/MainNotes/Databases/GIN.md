GIN расшифровывается как Generalized Inverted Index — это так называемый _обратный индекс_. Он работает с типами данных, значения которых не являются атомарными, а состоят из элементов. При этом индексируются не сами значения, а отдельные элементы; каждый элемент ссылается на те значения, в которых он встречается.  
  
Хорошая аналогия для этого метода — алфавитный указатель в конце книги, где для каждого термина приведен список страниц, где этот термин упоминается. Как и указатель в книге, индексный метод должен обеспечивать быстрый поиск проиндексированных элементов. Для этого они хранятся в виде уже знакомого нам [B-дерева](https://habrahabr.ru/company/postgrespro/blog/330544/) (для него используется другая, более простая, реализация, но в данном случае это несущественно). К каждому элементу привязан упорядоченный набор ссылок на строки таблицы, содержащие значения с этим элементом. Упорядоченность не принципиальна для выборки данных (порядок сортировки TID-ов не несет в себе особого смысла), но важна с точки зрения внутреннего устройства индекса.