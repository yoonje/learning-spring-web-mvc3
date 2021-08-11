# Spring Framework 정리 자료
실전! 스프링 MVC 2편 정리 문서

Table of contents
=================
<!--ts-->
   * [메시지, 국제화](#메시지-국제화)
   * [검증1](#검증1)
   * [검증2](#검증2)
   * [로그인 처리1](#로그인-처리1)
   * [로그인 처리2](#로그인-처리2)
   * [예외 처리와 오류 페이지](#예외-처리와-오류-페이지)
   * [API 예외 처리](#API-예외-처리)
   * [스프링 타입 컨버터](#스프링-타입-컨버터)
   * [파일 업로드](#파일-업로드)
<!--te-->

메시지 국제화
=======

### 메시지와 국제화

##### 메시지
- 다양한 메시지를 `messages.properteis` 한 곳에서 관리하도록 하는 기능으로 HTML, 자바 소스 등에서 사용할 수 있음
- messages.properties
```properties
item=상품 
item.id=상품 ID 
item.itemName=상품명 
item.price=가격 
item.quantity=수량
```
- html
```html
<label for="itemName" th:text="#{item.itemName}"></label>
```

##### 국제화
- 메시지에서 설명한 메시지 파일(messages.properteis)을 각 나라별로 별도로 관리하면 서비스를 국제화할 수 있음
- HTTP accept-language 해더 값을 사용하거나 사용자가 직접 언어를 선택하도록 하고, 쿠키 등을 사용해서 처리
- 국제화 기능을 적용하려면 messages_en.properties, messages_ko.properties 와 같이 resources 밑에 파일명 마지막에 언어 정보를 함께 둬서 처리
- messages_en.properties
```
item=Item 
item.id=Item ID
item.itemName=Item Name 
item.price=price 
item.quantity=quantity
```
- messages_ko.propertis
```
item=상품 
item.id=상품 ID 
item.itemName=상품명 
item.price=가격 
item.quantity=수량
```

### 스프링 메시지 소스

##### 메시지 소스 등록
- 메시지 관리 기능을 사용하려면 스프링이 제공하는 MessageSource를 스프링 빈으로 등록 해야하므로 기본 구현체인 ResourceBundleMessageSource를 등록
- 기본으로 `spring.messages.basename=messages`로 프로퍼티에 메시지 파일만 등록이 되지만 국제화를 위해서는 따로 작업해야함
- 빈으로 등록하는 방법
  - Java
```java
@Bean
public MessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasenames("messages", "errors"); // messages.properties
    messageSource.setDefaultEncoding("utf-8"); // 인코딩
    return messageSource;
}
```
  - application.properties로 등록하는 방법
```yml
spring.messages.basename=messages,config.i18n.messages
```

##### 메시지 소스로 메시지 사용
- MessgaeSource 인터페이스를 보면 코드를 포함한 일부 파라미터로 메시지를 읽어오는 기능을 제공
- 메시지 읽어오기
```java
@Autowired
private MessageSource ms;

@Test
void helloMessage() {
    String result = ms.getMessage("hello", null, null); // code, args, locale 
    assertThat(result).isEqualTo("안녕");
}
```
- 메시지가 없는 경우를 고려해서 메시지 읽어오기
```java
@Autowired
private MessageSource ms;

@Test
void notFoundMessageCodeDefaultMessage() {
    String result = ms.getMessage("no_code", null, "기본 메시지", null); // code, args, default, locale 
    assertThat(result).isEqualTo("기본 메시지"); 
}
```
- 매개변수를 사용해서 메시지 읽어오기
```java
@Autowired
private MessageSource ms;

@Test
void argumentMessage() {
    String result = ms.getMessage("hello.name", new Object[]{"Spring"}, null); // code, args, locale 
    assertThat(result).isEqualTo("안녕 Spring");
}
```
- 국제화 파일 선택해서 메시지 읽기
```java
@Test
void defaultLang() {
    assertThat(ms.getMessage("hello", null, null)).isEqualTo("안녕"); 
    assertThat(ms.getMessage("hello", null, Locale.KOREA)).isEqualTo("안녕");
}
```
##### 타임리프로 메시지 사용
- 타임리프의 메시지 표현식 #{...} 를 사용하면 스프링의 메시지를 편리하게 조회
- 메시지 파라미터 없는 경우 메시지 사용
```html
<th th:text="#{label.item.id}">ID</th>
```
- 메시지 파라미터 있는 경우 메시지 사용
```html
<p th:text="#{hello.name(${item.itemName})}"></p>
```

### 웹 애플리케이션에 적용하기

##### 웹 애플리케이션에 적용하는 방법
- 국제화된 다국어 파일만 있으면 웹 애플리케이션에 다국어가 적용됨
- Locale 정보를 알아야 언어를 선택할 수 있는데, 스프링은 언어 선택시 기본으로 `Accept-Language 헤더`의 값을 사용
- 웹 브라우저의 언어 설정 값을 변경하면 요청시 Accept-Language 의 값이 변경

##### LocaleResolver
- 스프링은 Locale 선택 방식을 변경할 수 있도록 LocaleResolver 라는 인터페이스를 제공하는데, 스프링 부트는 기본으로 Accept-Language 를 활용하는 AcceptHeaderLocaleResolver 를 사용
- Locale 선택 방식을 변경하려면 LocaleResolver 의 구현체를 변경해서 쿠키나 세션 기반의 Locale 선택 기능을 사용

검증1
=======


검증2
=======


로그인 처리1
=======


로그인 처리2
=======


예외 처리와 오류 페이지
=======


API 예외 처리
=======


스프링 타입 컨버터
=======


파일 업로드
=======
