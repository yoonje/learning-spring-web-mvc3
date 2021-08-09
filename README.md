# Spring Framework 정리 자료
실전! 스프링 MVC 2편 정리 문서

Table of contents
=================
<!--ts-->
   * [티임리프1](#타임리프1)
   * [티임리프2](#타임리프2)
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

타임리프1
=======
### 타임리프
##### 티임리프 특징
- 서버 사이드 HTML 렌더링 (SSR): 백엔드 서버에서 HTML을 동적으로 렌더링 하는 용도
- 네츄럴 템플릿: 순수 HTML을 최대한 유지
- 스프링 통합 지원

##### 타임리프 기본 표현식
- 간단한 표현
  - 변수 표현식: ${...}
  - 선택 변수 표현식: *{...}
  - 메시지 표현식: #{...}
  - 링크 URL 표현식: @{...}
  - 조각 표현식: ~{...}
- 리터럴
  - 텍스트: 'one text', 'Another one!',...
  - 숫자: 0, 34, 3.0, 12.3,...
  - 불린: true, false
  - 널: null
  - 리터럴 토큰: one, sometext, main,...
- 문자 연산
  - 문자합치기: +
  - 리터럴 대체: |The name is ${name}|
- 산술 연산
  - Binary operators: +, -, *, /, %
  - Minus sign (unary operator): -
- 불린 연산
  - Binary operators: and, or
  - Boolean negation (unary operator): !, not
- 비교와 동등 연산
  - 비교:>,<,>=,<=(gt,lt,ge,le)
  - 동등 연산: ==, != (eq, ne)
- 조건 연산
  - If-then: (if) ? (then)
  - If-then-else: (if) ? (then) : (else)
  - Default: (value) ?: (defaultvalue)
- 특별한 토큰
  - No-Operation: _

### 텍스트
##### text
- HTML 테그의 속성에 기능을 정의해서 동작하는데, HTML의 콘텐츠(content)에 데이터를 출력할 때는 다음과 같이 th:text 를 사용
```html
<li>th:text = <span th:text="${data}"></span></li>
```
- HTML 테그의 속성이 아니라 `HTML 콘텐츠 영역`안에서 직접 데이터를 출력하고 싶으면 다음과 같이 `[[...]]` 를 사용
```html
<li>컨텐츠 안에서 직접 출력하기 = [[${data}]]</li>
```
##### utext
- 웹브라우저는 <를 HTML 테그의 시작으로 인식하여 특수 문자를 HTML 엔티티로 변경하여 이스케이프 처리하여 이스케이프 처리하지 않으려면 `utext`를 사용
```html
<li>th:utext = <span th:utext="${data}"></span></li>
```

### 변수 SpringEL
##### 변수 표현식
- 타임리프에서 변수를 사용할 때는 변수 표현식을 사용
```html
${..}
```
##### SpringEL을 통한 다양한 표현식 사용
- Object
  - user.username : user의 username을 프로퍼티 접근 
  - user['username'] : user의 username을 프로퍼티 접근
  - user.getUsername() : user의 getUsername() 을 직접 호출
- List
  - users[0].username : List에서 첫 번째 회원을 찾고 username 프로퍼티 접근
  - users[0]['username'] : List에서 첫 번째 회원을 찾고 username 프로퍼티 접근
  - users[0].getUsername() : List에서 첫 번째 회원을 찾고 메서드 직접 호출
- Map
  - userMap['userA'].username : Map에서 userA를 찾고, username 프로퍼티 접근
  - userMap['userA']['username'] : Map에서 userA를 찾고, username 프로퍼티 접근
  - userMap['userA'].getUsername() : Map에서 userA를 찾고 메서드 직접 호출

##### 지역 변수 선언
- th:with 를 사용하면 지역 변수를 선언해서 사용할 수 있는데 지역 변수는 선언한 테그 안에서만 사용
```html
<h1>지역 변수 - (th:with)</h1>
<div th:with="first=${users[0]}">
    <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p>
</div>
```

### 타임리프 기본 객체들
- #request
- #response
- #session
- #servletContext}
- #locale

### 타임리프 유틸리티 객체들
- #message : 메시지, 국제화 처리
- #uris : URI 이스케이프 지원
- #dates : java.util.Date 서식 지원 
- #calendars : java.util.Calendar 서식 지원 
- #temporals : 자바8 날짜 서식 지원
- #numbers : 숫자 서식 지원
- #strings : 문자 관련 편의 기능
- #objects : 객체 관련 기능 제공
- #bools : boolean 관련 기능 제공
- #arrays : 배열 관련 기능 제공
- #lists , #sets , #maps : 컬렉션 관련 기능 제공 
- #ids : 아이디 처리 관련 기능 제공, 뒤에서 설명

### URL 링크
- 타임리프에서 URL을 생성할 때는 @{...} 문법을 사용
- 단순 URL
```html
<li><a th:href="@{/hello}">basic url</a></li>
```
- 쿼리 파라미터 
```html
<li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
```
- 경로 변수
```html
<li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
```

### 리터럴
- 리터럴: 리터럴은 소스 코드상에 고정된 값을 말하는 용어
- 리터럴 종류
  - 문자: 'hello' (문자를항상 '로 감싸야함)
  - 숫자: 10
  - 불린: true , false 
  - null: null
```html
<li>'hello' + ' world!' = <span th:text="'hello' + ' world!'"></span></li>
```
- 리터럴 대체: ||를 사용하면 템플릿으로 리터럴을 대체할 수 있음
```html
<span th:text="|hello ${data}|">
```

### 연산
- 비교연산: HTML 엔티티를 사용할 때는 주의
  > (gt), < (lt), >= (ge), <= (le), ! (not), == (eq), != (neq, ne)
- 조건식: 자바의 조건식과 유사
```html
<li>1 > 10 = <span th:text="1 &gt; 10"></span></li>
<li>1 gt 10 = <span th:text="1 gt 10"></span></li>
<li>1 >= 10 = <span th:text="1 >= 10"></span></li>
<li>1 ge 10 = <span th:text="1 ge 10"></span></li>
<li>1 == 10 = <span th:text="1 == 10"></span></li>
<li>1 != 10 = <span th:text="1 != 10"></span></li>
```
- Elvis 연산자: 조건식의 편의 버전
```html
<li>(10 % 2 == 0)? '짝수':'홀수' = <span th:text="(10 % 2 == 0)? '짝수':'홀수'"></span></li>
```
- No-Operation: _ 인 경우 마치 타임리프가 실행되지 않는 것 처럼 동작
```html
<li>${data}?: _ = <span th:text="${data}?: _">데이터가 없습니다.</span></li>
```
### 속성값 설정


타임리프2
=======


메시지 국제화
=======


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
