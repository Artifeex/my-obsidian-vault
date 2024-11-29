### Dockerfile
Пишем руками Dockerfile.
### Buildpacks
Используем плагин в maven, который на основе нашего проекта сгенерит dockerfile.
Он по умолчанию у нас есть, т.к. когда мы генериуем проект, используя spring-io, то добавляется плагин spring-boot, внутри которого есть вот такая команда, которая и создаст нам docker image, используя Buildpacks.
![[Pasted image 20241129155233.png]]
### Google Jib
Логика аналогичная Buildpacks, но поддерживается гуглом.
mvn compile jib:dockerBuild
```xml
<plugin>  
    <groupId>com.google.cloud.tools</groupId>  
    <artifactId>jib-maven-plugin</artifactId>  
    <version>3.4.4</version>  
    <configuration>       <to>  
          <image>artifeexs/${project.artifactId}:s4</image>  
       </to>  
    </configuration>  
</plugin>
```
