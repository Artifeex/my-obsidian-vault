equals и hashCode - это стандартные методы класса Object, которые мы можем переопределять.
И если мы хотим использовать наш объект внутри HashMap, то нужно реализовать методы equals и hashCode и контракт между ними, чтобы все работа корректно.

Если equals() для двух объектов вернуло true, то, значит, у этих объектов должны быть одинаковые hashCode. Но при этом в обратную сторону не действует. Если у нас есть 2 объекта, которые имеют одинаковый hashCode, это не значит, что эти два объекта равны.

Например, если у нас два объекта, которые равны по equals, но при этом у них разных hashCode. То в HashMap, из-за того, что разные hashCode эти два объекта будут находиться в разных бакетах. Но при этом значения у них равны. А это, значит, что мы храним дубликаты в нашей Map.

### Контракт метода equals
![[Pasted image 20241112102742.png]]
![[Pasted image 20241112102755.png]]
![[Pasted image 20241112102806.png]]
![[Pasted image 20241112102817.png]]
### Контракт hashCode
![[Pasted image 20241112102838.png]]

