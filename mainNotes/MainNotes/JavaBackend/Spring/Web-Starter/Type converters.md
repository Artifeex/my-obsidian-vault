У нас есть проблема. Нам нужно уметь конвертировать дату, переданную в виде строки "2000-04-20" в LocalDate. Для этого в application.yaml мы включаем iso стандарт для даты.
![[Pasted image 20240913110634.png]]
После этого mockMvc тест на создание проходит.
![[Pasted image 20240913110746.png]]