![[Pasted image 20241224154828.png]]
![[Pasted image 20241224161713.png]]
SpringSecurity обрабатывает только 2 типа исключений(плюс их потомки):
- AuthenticationException(статус 401)
- AccessDeniedException(статус 403)
Возникающие исключение сначала попадают в ExceptionTranslationFilter, который смотрит на тип исключение и отправляет это исключение в соответсвующи обработчик:
- AuthenticationEntryPoint
- AccessDeniedHandler

Напишем свой кастомный AuthenticationEntryPoint:
![[Pasted image 20241224163920.png]]
А также, чтобы Spring Security знал, какой нужно использовать AuthenticationEntryPoint в случае AuthenticationException мы должны в configuration указать это. Например, пусть наш кастомный AuthenticationEntyPoint срабатывает для Basic аутентификации. 
![[Pasted image 20241224164052.png]]

А вообще есть множество реализаций AuthenticationEntryPoint, которые Spring предоставляет по умолчанию. Например, LoginUrlAuthenticationEntryPoint - срабатывает, когда у нас включен formLogin и человек пытается зайти на защищенную страничку, тем самым возникает AuthenticationException и вызывается этот LoginUrlAuthenticationEntryPoint, который и выполняет редирект на страничку логина.
![[Pasted image 20241224164254.png]]

Также внутри commence методы мы можем преобразовать body, которое вернется. Но нужно в response.setStatus указывать, а не sendError, т.к. если укажем sendError, то спринг автоматически обрабатывает такой error и заполнит body дефолтным текстом, а не тем, что мы передали. 
![[Pasted image 20241224165600.png]]

### Важное замечание
Переопределять AuthenticationEntryPoint есть смысл только для basic authentication. Для formAuthentication такое Spring запрещает. Если мы хотим как-то закастомить путь, на который будет редирект или еще что-либо при работе с formLogin, то есть другие варианты конфигурации, а не определение нового AuthenticationEntryPoint.

Вот различные методы, которыми мы можем настроить. При этом у нас даже не будет метода для определения своего класса AuthenticationEntryPoint для formLogin.
![[Pasted image 20241224170337.png]]
### Глобальные настройки для exceptionHandling
![[Pasted image 20241224170449.png]]
Зачем оно надо? Когда мы написали httpBasic() - и добавили для него AuthenticationEntryPoint, то мы определили обработку ошибок аутентификации только внутри httpBasic flow(т.е. во время логина). Но AuthenticationException могла возникнуть в любой части нашего приложения. И чтобы мы были готовы к такой ситуации, мы и создаем глобальный AuthenticationEntryPoint. 

### Custom AccessDeniedHandler
Создаем:
![[Pasted image 20241224172326.png]]
Добавляем в Configuration. Причем добавляем глобально, т.к. нельзя добавить отдельно для httpBasic и отдельно для formLogin.
![[Pasted image 20241224172316.png]]
Дополнительно можем указать deniedPage. С помощью этой настройки указывается на какой путь будет перенаправлен пользователь в случае 403 ошибки. 
![[Pasted image 20241224172656.png]]

Но вообще, все вот эти доп настройки с formLogin и т.д. - это все нужно чисто, когда с нами общаются и ожидают от нас HTML странички(т.е. у нас Spring MVC приложение). Если с нами общаются по REST, то используется basicAuthentication(ну или другой вид, когда пройдем OAuth2 и JWT).

