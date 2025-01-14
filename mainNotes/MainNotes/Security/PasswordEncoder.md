![[Pasted image 20241224113124.png]]
Мы не можем хранить наши пароли в не захэшированном виде, поскольку если кто-либо получить доступ к дампу памяти нашего приложение или к таблице в БД, то он сразу узнает все пароли. Поэтому и используется PasswordEncoder.

Создаем бин PasswordEncodera. Реализация этого интерфейса довольно много. По умолчанию используется BCryptPasswordEncoder.
![[Pasted image 20241223122710.png]]

![[Pasted image 20241223122811.png]]

Есть интересный метод у интерфейса PasswordEncoder. Он возвращает true, если хэш пароля нужно прогнать еще раз через хэширование, чтобы увеличить защиту. 
![[Pasted image 20241224113708.png]]

### Рассмотрим основные реализации интерфейса PasswordEncodera
#### NoOpPaswordEncoder
Нельзя использовать в продакшене. Используется только для каких-то демо приложений, если мы хотим просто использовать PasswordEncoder интерфейс, но прим этом пароли храним в сыром виде.
#### StandardPasswordEncoder(deprecated)
Использует SHA-256 hashing algorithm. Использовался раньше, теперь существует только ради обратной совместимости. Стал депрекейтед из-за того, что sha-256 функция вычисляется довольно быстро, а мы говорили, что нам нужна функция, которая намеренно замедлена. 
#### Pbkdf2PasswordEncoder
![[Pasted image 20241224114456.png]]
iterations - сколько раз алгоритм будет применяться. Т.е. посчитали хэш функцию от сырого пароля, потом этот результат снова в хэш функцию и получаем другой хэш и т.д. Т.е. чем больше итераций, тем дольше по времени вычисляется итоговый хэш пароля. 

Также он интересен тем, что мы можем передать в него secret и тогда пароль станет еще безопаснее хранится, поскольку теперь для вычисления хэша нужно: соль + секрет + пароль. И тогда, даже если хакер получит доступ к БД, где будут хранится хэши, но при этом у него не будет доступа к секрет, то он никак не сможет взломать даже брутфорсом. 

Но лучше такое не использвать, т.к. появляется дополнительная сложность и отвественность с хранением секрета. А также сам алгоритм хэширования в этом Encodere старый. 

#### BCrypt Password Encoder
Использует алгоритм BCrypt.
![[Pasted image 20241224115215.png]]
Есть 3 версии:
- 2a
- 2b
- 2y
Также можем настраивать у него силу. И чем больше сила, тем больше раундов хэширования будет происходить.

#### SCrypt Password Encoder
Продвинутая версия BCryptPasswordEncoder. Используем SCrypt функцию. Особенностью является то, что мы можем более гибко настраивать какие ресурсы необходимы для вычисления функции! 

#### Argon2PasswordEncoder
Использует Argon2 hashing function. Является по сути самой последний версией SCrypt функции. 

В реальных проектах, чаще всего используется BCrypt Password Encoder. Т.к. настройка SCrypt и Argon требует дополнительных усилий, а также замедляет процесс авторизации пользователей. (Хотя в банках, наверное, все-таки такие функции используются или что-нибудь самописное).

### Важное замечание по созданию Password Encoder в Spring

![[Pasted image 20241224121035.png]]
Лучше использовать DelegatingPasswordEncoder, а не создавать бин конкретного passwordEncodera. Он поможет в том случае, если в будущем мы захотим перейти на какой-то другой алгоритм шифрования. Он смотрит на ту запись в БД, где хранится хэш пароля, извлекает из нее {bcrypt} - вот такой префикс, по которому он понимает, что за Encoder был использован и для нашей ситуации будет использован именно нужный PasswordEncoder. И если в будущем, мы переедем на новый алгоритм шифрования, то старые пользователи будут все также использовать свои пароли и для их паролей будет использоваться тот хэширующий алгоритм, который был использован в тот момент, когда они регистрировались, а для новых пользователей уже будет использован новый алгоритм.

Кратко: Помогает поддерживать сразу несколько видов encoder-ов в рамках одного приложения и БД.

