Идея состоит в том, чтобы спроецировать(смаппить) сущность из БД на [[DTO]] объект этой сущности. Например, нам не нужно, чтобы вся информация о пользователе гуляла по приложению. Нам нужны только некоторые поля, а не все поля пользователя, поэтому мы и создаем DTO.
Создадим DTO для пользователя
![[Pasted image 20240909095547.png]]

После этого в Repository мы можем использовать это DTO! Ничего настраивать не нужно. Он сам смаппит наши Entity из БД на PersonalInfo. Единственное, нужно, чтобы название полей в DTO были такими же, как и у Entity. Можем сразу использовать PersonalInfo в Repository.
![[Pasted image 20240909095903.png]]

Но можно еще делать и Generic маппинг. Т.е. чтобы не пладить множество методов, которые будут маппить различные сущности на DTO можно параметризовать и все будет также работать. Нужно будет только передать класс DTO при использовании.
![[Pasted image 20240909100051.png]]
Передаем PersonalInfo.class. 
![[Pasted image 20240909100148.png]]

На практике проекции нужны чаще всего при выполнении нативных запросов. Но в таком случае для DTO не работают классы, нужно создавать интерфейсы для DTO.
И т.к. это интерфейс, то мы не можем создавать поля. Вместо этого мы создаем getters для потенциальных полей. Название возвращаемого столбца из БД должно соответствовать названию из getter метода. Т.е. когда мы делаем getFirstname из БД должен вернуться firstname столбец. 
![[Pasted image 20240909100424.png]]

Здесь мы для birth_date сделали alias, т.к. в БД этот столбец имеет название birth_date, а нам, чтобы смаппить на Projection нужно, чтобы он имел название birthDate как в интерфейсе в геттере.
![[Pasted image 20240909100810.png]]

После этого можем спокойно использовать и через getter методы получать значения полей. 
![[Pasted image 20240909101113.png]]

Также можно добавлять некоторый функционал, например, хотим иметь возможность получать FullName, но такого поля у нас в БД нет. Тогда используем SpEL. С помощью target обращаемся к объекту PersonalInfo2.
![[Pasted image 20240909101309.png]]

Использование:
![[Pasted image 20240909101459.png]]

И проекции прикольно использовать, когда мы хотим нативно использовать какие-то агрегирующие функции внутри БД и потом просто смаппить значение из функции на поле в проекции. 