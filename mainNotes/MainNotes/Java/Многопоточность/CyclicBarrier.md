Отличие от CountDownLatch в том, что здесь есть только метод await() и потоки, которые вызвали await() блокируются до тех пор, пока заранее заданное количество потоков не вызвало тоже await().
![[Pasted image 20241112132750.png]]![[Pasted image 20241112132801.png]]
Удобно воспользоваться Cached, который создаст ровно столько потоков, сколько нужно, чтобы все потоки не зависли:
![[Pasted image 20241112132825.png]]После разблокировки в await() потоки продолжают свое выполнение.

### Не забываем, что все, что есть в java.util.concurrent использует Unsafe для реализации потокобезопасности.