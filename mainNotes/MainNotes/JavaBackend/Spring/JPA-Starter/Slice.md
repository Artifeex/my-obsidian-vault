Наследник Streamable из-за этого много методов, которые похожи на работу со Stream(повторяют даже). Основные элементы:
* number - номер текущей странички. Номер слайса.
* size - размер(limit) странички.
* numberOfElements - актуальное количество элементов в слайсе. Хоть мы и сделали limit, но могло вернуться, например, только 3 элемента. А limit =10. Тогда numberOfElements = 3.
Использование:
![[Pasted image 20240908171612.png]]
Со Slice можем работать как со страничками. Мы получили первый slice элементов, можем с ними как-то поработать, потом с помощью hasNext(), проверить, есть ли в выборке еще slice элементов. Если есть, то можем получить этот новый slice элементов, используя еще один вызов метода из Repository, передав Pageable, у которого просто offset на 1 страничку больше, чтобы мы пропустили те элементы слайса, которые получили при прошлом слайсе.
![[Pasted image 20240908171856.png]]

И это используется при пагинанации на сайте! Но на самом деле у Slice есть 1 минус, мы не можем с помощью него узнать, сколько всего страничек будет. Например, на скрине снизу мы видим цифру 20, которая говорит, что есть 20 страничек. Но в Slice такого функционала нет. Поэтому нам нужен Page, который дополнительно делает запрос count, который узнает, сколько всего строк в таблице. И с помощью этого count запроса мы и можем узнать, сколько всего страниц, разделив количество строк в таблице на размер одной страницы. Таким образом, если нам нужен такой счетчик, который покажет, сколько всего страниц, то используем Page, а если не нужен, то Slice.
![[Pasted image 20240908172244.png]]