Теперь в тестах при тестировании контроллеров нам также нужно пройти аутентификацию.
Для решения этой проблемы нужно добавить зависимость
![[Pasted image 20240916223529.png]]
И уже в методе тестирования просто в SecurityContextHolder добавить наш SecurityContext, в который добавим нашего пользователя.
![[Pasted image 20240916223515.png]]
Но удобнее использовать второй способ, который использует аннотации @WithMockUser
![[Pasted image 20240916223802.png]]
ЧТобы не ставить над каждым тестовым методом мы ставим эту аннотацию над всем классом тестов.
Т.е. мы так всем методам дадим какого-то дефолтного тестового пользователя. А если вдруг в каком-то тесте нам нужен какой-то другой юзер, то просто над этим тестовым методом используем аннотацию @WithMockUser, которая будет главнее той, что находится над всем тестовым классом.
