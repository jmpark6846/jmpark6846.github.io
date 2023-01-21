---
title: "Spring Security와 @WebMvcTest 사용시 403 에러 발생"
date: 2022-12-26T20:20:26+09:00
draft: false
summary: "ContextConfiguration으로 설정을 매뉴얼하게 불러오기"
tags: ["Spring", "테스트"]
isCJKLanguage: true
---

# web 


* WebMvcTest로 POST endpoint에 대한 테스트 작성 후 실행하자 별다른 문제가 없었음에도 201이 아닌 403 발생
* Spring Security를 사용하면 자동으로 권한 체크를 하도록 되어있는 것 같았음
* 우선 간단하게 Security Configuration 작성(REST API만 개발할 예정이라 CSRF, 세션을 사용하지 않았다.)

```java
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{

        return http.httpBasic().disable()
                .csrf().disable()
                .authorizeHttpRequests()
                .anyRequest()
                .permitAll()
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and().build();
    }
}

```

* 테스트 실행 후 여전히 403 에러 발생

> @WebMvcTest  
> Using this annotation will disable full auto-configuration and instead apply only configuration relevant to MVC tests (i.e. @Controller, @ControllerAdvice, @JsonComponent, Converter/GenericConverter, Filter, WebMvcConfigurer and HandlerMethodArgumentResolver beans but not @Component, @Service or @Repository beans).

* 정확히는 모르겠지만 @WebMvcTest가 @Configuration 파일을 import 하지 않는 것 같다.
* 테스트 케이스에 @ContextConfiguration 적용하여 설정 파일 로드
```java
@WebMvcTest(TicketController.class)
@ContextConfiguration(classes = { SecurityConfiguration.class, })
```
* 테스트 실행하니 404 에러 발생 -> Controller 가 제대로 Import 되지 않는 것 같다.
* @Import 사용하여 직접 컨트롤러 불러오기
```java
@WebMvcTest
@Import(TicketController.class)
@ContextConfiguration(classes = { SecurityConfiguration.class, })
```
* 정상 작동

