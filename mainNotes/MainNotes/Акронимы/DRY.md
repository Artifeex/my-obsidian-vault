Dont Repeat Yourself

В проектировании DRY тоже имеет место — доступ к конкретному функционалу должен быть доступен в одном месте, унифицирован и сгруппирован по какому-либо принципу, а не «разбросан» по системе в произвольных вариациях.

Если код не дублируется, то для изменения логики достаточно внесения исправлений всего в одном месте и проще тестировать одну (пусть и более сложную) функцию, а не набор из десятков однотипных.

В рамках небольших проектов следовать принципу DRY не так уж и сложно, но в рамках, поскольку разработчики знают кодовую базу. В больших же проектах, разработчик может даже не знать о том, что такая функция уже была где-то реализована. 

Следование принципу DRY всегда приводит к декомпозиции сложных алгоритмов на простые функции. А декомпозиция сложных операций на более простые (и повторно используемые) значительно упрощает понимание программного кода. Приводит к декомпозиции, поскольку какую-то большую функцию делят на более маленькие и потом эти маленькие также могут быть переиспользованы.