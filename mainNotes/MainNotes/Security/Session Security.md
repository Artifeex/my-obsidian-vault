Сессия имеет дефолтный timeout в виде 30 минут с момента создания(создается, когда пользователь логинится). Когда это время проходил, то если пользователь попытается получить доступ к какому-то защищенному ресурсу, его снова перекинет на страницу логина.
После логина создается куки JSESSIONID:
![[Pasted image 20241224173455.png]]
Чтобы настроить timeout:
![[Pasted image 20241224173558.png]]

### Concurrent Session Control
По дефолту спринг никак не ограничивает количество открытых сессий пользователя. 
![[Pasted image 20241224175200.png]]

Но единичку ставят очень редко, по крайней мере, в QA envs точно, поскольку один и тот же тестер может работать в нескольких браузерах, приложения(postman) и все это на одном пользователе, а каждое такое приложение может требовать сессию.

Мы получим вот такую ошибку на самой первой сессии, если кто-то зайдет в новой сессии. Т.е. мы сначала в POSTMAN залогинились у нас сохранилась кука JSESSIONID. Потом мы залогинились в браузере, там тоже сохранилась кука JSESSIONID, но после этого, если мы в POSTMAN отправим запрос, то там будет session has been expired:
![[Pasted image 20241224175545.png]]

Мы можем изменить это поведение и сделать наоборот, чтобы, если уже есть созданная сессия, то если пользователь пытается залогиниться при живой сессии с нового устройства, то у него не получится и будет такая же ошибка, но уже при логине. 
Для этого добавляется следующая настройка
![[Pasted image 20241224175938.png]]
После этой настройки имеет вот такое поведение: когда пользователь пытается залогиниться еще раз, то не старая сессия убьется, а новая не создается:
![[Pasted image 20241224180115.png]]

Итог:
![[Pasted image 20241224180428.png]]

### Session Hijacking, Session Fixation
Session Hijacking - это когда злоумышленник может украсть JSESSIONID. Украсть он может во время передачи данных по сети(чтобы избежать этого - нужно использовать HTTPS), а также если получил доступ к компьютеру пользователя или данных браузера.

Session Fixation
![[Pasted image 20241225092917.png]]
Spring по умолчанию защищен от Session Fixation атаки:
![[Pasted image 20241225093008.png]]
changeSessionId - это стратегия в Spring по умолчанию.
