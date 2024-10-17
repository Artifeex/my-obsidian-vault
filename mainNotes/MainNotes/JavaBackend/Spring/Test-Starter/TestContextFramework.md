
![[Pasted image 20240905193056.png]]
В Spring реализован свой Framework для написания интеграционных тестов. Для этого нужно использовать SpringExtension, подключив с помощью @ExtendWith и внедрившись в [[Жизненный цикл тестов]].
Для класса теста, который использует SpringExtension создается TestContextManager. Внутри него есть 2 основных поля.
- TestContext - в нем можно получить информацию о том, в каком классе запущен тест, какой тест запущен(метод), получить instance тестового класса(он же создается для каждого теста новый). И самое главное - это получение applicationContext! А из него мы можем получать наши бины для внедрения зависимостей в нашем тесте. И за счет этого реализован механизм кэширования applicationContext.
- TestExecutionListeners - TestContextManager делегирует те коллбеки из [[Extension Model]], которые мы видели на вызов такого метода у всех сохраненных внутри TestExecutionListeners. Т.е. просто для каждого метода из ExtensionModel он вызывает соответствующий метод из TestExecutionListeners для каждого листенера.
Внутренности TestContextManager:![[Pasted image 20240905202032.png]]
SpringExtension:
![[Pasted image 20240905194050.png]]
Внутри вызова метода beforeEach внутри SpringExtension. Происходит проксирование вызова на TestContextManager. 
![[Pasted image 20240905194754.png]]

Внутри вызова TestContextManager.beforeTestExecution:
![[Pasted image 20240905194642.png]]

