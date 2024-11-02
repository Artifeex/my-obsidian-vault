Spring 3.0.2
```java
@Bean  
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
    http  
            .csrf().disable()  
            .cors()  
            .and()  
            .httpBasic().disable() //отключаем basic аутентификацию  
            .sessionManagement()  
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)  
            .and()  
            .exceptionHandling()  
            .authenticationEntryPoint((request, response, authException) -> {  
                response.setStatus(HttpStatus.UNAUTHORIZED.value());  
                response.getWriter().write("Unauthorized"); //если аутентификация проходит неуспешно, то выбрасывается исключение и срабатывает entyPoint, который мы написали  
            })  
            .accessDeniedHandler((request, response, accessDeniedException) -> {  
                response.setStatus(HttpStatus.FORBIDDEN.value()); //когда будет попытка доступа к запрещенному ресурсу  
                response.getWriter().write("Forbidden");  
            })  
            .and()  
            .authorizeHttpRequests()  
            .requestMatchers("/api/v1/auth/**").permitAll() //должны дать возможность получать и обновлять токены  
            .requestMatchers("/swagger-ui/**").permitAll()  
            .requestMatchers("/v3/api-docs/**").permitAll()  
            .anyRequest().authenticated() //все остальные запросы надо проверять  
            .and()  
            .anonymous().disable() // отключаем возможность анонимного посещения  
            //добавляем наш фильтр, который будет идти после UsernamePasswordAuthenticationFilter(а этот фильтр проводит аутентификацию)            //и в SecurityContext кладет Authentication. А потом уже мы и будем добавлять токены в нашем фильтре            .addFilterBefore(new JwtTokenFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);  
  
            return http.build();  
  
}
```
Spring 3.1.2:
```java
@Bean  
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
    http  
            .csrf(AbstractHttpConfigurer::disable)  
            .cors(AbstractHttpConfigurer::disable)  
            .httpBasic(AbstractHttpConfigurer::disable)  
            .sessionManagement(sessionManagement -> sessionManagement.sessionCreationPolicy(SessionCreationPolicy.STATELESS))  
            .exceptionHandling(configurer ->  
                    configurer.authenticationEntryPoint((request, response, authException) -> {  
                                response.setStatus(HttpStatus.UNAUTHORIZED.value());  
                                response.getWriter().write("Unauthorized"); //если аутентификация проходит неуспешно, то выбрасывается исключение и срабатывает entyPoint, который мы написали  
                            })  
                            .accessDeniedHandler((request, response, accessDeniedException) -> {  
                                response.setStatus(HttpStatus.FORBIDDEN.value()); //когда будет попытка доступа к запрещенному ресурсу  
                                response.getWriter().write("Forbidden");  
                            }))  
            .authorizeHttpRequests(configurer ->  
                    configurer.requestMatchers("/api/v1/auth/**").permitAll() //должны дать возможность получать и обновлять токены  
                            .requestMatchers("/swagger-ui/**").permitAll()  
                            .requestMatchers("/v3/api-docs/**").permitAll()  
                            .anyRequest().authenticated()) //все остальные запросы надо проверять  
            .anonymous(AbstractHttpConfigurer::disable)// отключаем возможность анонимного посещения  
                        //добавляем наш фильтр, который будет идти после UsernamePasswordAuthenticationFilter(а этот фильтр проводит аутентификацию)  
            //и в SecurityContext кладет Authentication. А потом уже мы и будем добавлять токены в нашем фильтре            .addFilterBefore(new JwtTokenFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);  
  
    return http.build();  
  
}
```
