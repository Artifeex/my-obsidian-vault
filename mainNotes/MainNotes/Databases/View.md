Они не являются таблицами, они являются представлениями таблиц(поэтому в них нельзя вставить, обновить или удалить). Их предназначение - делать их них какие-то выборки, т.е. SELECT. Но мы могли делать SELECT из обычных таблиц, зачем нам тогда View ? Они нужны для того, чтобы если у нас был какой-то сложный запрос из которого мы хотим получать разные данные - т.е. писать разные выборки внутри SELECT, то нам не приходилось повторять этот сложный запрос, а ссылаться на этот View и выбирать данные из него. Т.е. View делаются на основании какого-то запроса.
Синтаксис: 
```SQL
CREATE VIEW name_view AS ЗАПРОС;
```
После этого появляется папочка Views, в которой будут располагаться созданные нами View.
![[Pasted image 20240927121755.png]]

После этого можем во FROM обращаться к этой View по имени:
![[Pasted image 20240927121831.png]]
Каждый раз, когда мы обращаемся к такой view, она каждый раз делает тот запрос, который мы указали при ее создании.

Если же мы хотим закешировать такой запрос, т.е., чтобы он не выполнялся каждый раз при обращении к View, то существует Materialized view.

### Materialized View
![[Pasted image 20240927121928.png]]
Создается также папочка
![[Pasted image 20240927121953.png]]
Т.е. теперь эта View уже хранит не запрос, который нужно сделать, чтобы получить данные, а сами данные, но все равно, вставлять, удалять и обновлять данные в ней нельзя.

Важным отличием между View и Materialized View является то, что если мы добавим какие-то данные в таблицы, которые участвовали при создании View, то обычное View узнает об этом(оно же просто новый запрос делает, а не кеширует данные и не хранит их на диске, как это делает Materialized View). А вот если мы обратимся к Materialized View, новые данные не будут отображены. Нам нужно обновить значения в Materialized View.

View - обычный подзапрос, который мы не видим.
Чтобы обновить Materialized view используем:
`REFRESH MATERIALIZED VIEW name_view;`
