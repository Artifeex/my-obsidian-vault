Параметр, который мы дополнительно можем передавать в запросы. Причем он работает со всеми типами запросов. И с @Query и c [[PartTreeQuery]]. Используется для того, чтобы динамически передавать в запрос сортировку, limit и offset.
![[Pasted image 20240908165914.png]]
![[Pasted image 20240908170049.png]]
Таким образом, с помощью pageable мы можем задавать limit и offset.

### Возможные возвращаемые значение при использовании Pageable
- List
- Stream
- Streamable - довольно редко используется на практике. Является реализацией Iterable интерфейса. Вместо него использует Slice.
- Slice
- Page
### [[Slice]]

### [[Page]]
