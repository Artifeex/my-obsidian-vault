Используется для маппинга HTTP запроса на метод контроллера, который обработает данный запрос.
![[Pasted image 20240910122844.png]]
Состоит из:
- value - path из URL, который обрабатывает метод.
- method - указывается [[HTTP методы|HTTP метод]].
- params - параметры запроса.
- headers - можем явно указывать.
- consumes - Content-Type, который мы можем обрабатывать на стороне сервера.
- produces - это Content-Type, который мы возвращаем из нашего сервера.