Внешний ключ используется для установки связи между таблицами в реляционных БД. 
Есть 2 основных способа задания:
1. Используем сразу же при создании поля ![[Pasted image 20240927104709.png]]
2. Используем в конце таблицы, когда задаем ограничения![[Pasted image 20240927104729.png]]

### Установка стратегий поведения при удалении строки, на которую ссылаются FOREIGN KEY
При создании FOREIGN KEY можем устанавливать правила по которым будет определяться, что будет происходить при удалении строки, на которую мы ссылаемся внешним ключом. Для этого используется ON DELETE.
![[Pasted image 20240927110831.png]]
- CASCADE - удаление строки влечет за собой удаление и всех строк, которые ссылались на эту строку с помощью FOREIGN KEY
- NO ACTION, RESTICT - не даст удалить.
- SET DEFAULT - установит какое-то дефолтное значения для нашего FOREIGN KEY после удаления.
- SET NULL - установить FOREIGN KEY = null.

![[Pasted image 20240927141549.png]]

### Cсылаться при создании внешнего ключа можно на любое уникальное поле другой таблицы, а не только на id