# Spring Framework 정리 자료
실전! 스프링 MVC 2편 정리 문서

Table of contents
=================
<!--ts-->
   * [메시지, 국제화](#메시지-국제화)
   * [검증](#검증)
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

검증
=======

### Bean Vlidation 소개
- Bean Validation을 잘 활용하면, 애노테이션 하나로 검증 로직을 매우 편리하게 적용r 가가능능
- Bean Validation은 특정한 구현체가 아니라 인터페이스 모음으로 Bean Validation 2.0(JSR-380)이라는 기술 
- Bean Validation을 구현한 기술중에 일반적으로 사용하는 구현체는 하이버네이트 Validator
  - jakarta.validation-api: Bean Validation 인터페이스 
  - hibernate-validator 구현체
- 스프링은 검증기부터 어노테이션까지 잘 통합되어 있음
### Bean Vlidation - 필드 오류
- 모델
```java
@Data
public class Item {

      private Long id;

      @NotBlank
      private String itemName;
      
      @NotNull
      @Range(min = 1000, max = 1000000)
      private Integer price;
      
      @NotNull
      @Max(9999)
      private Integer quantity;
      
      public Item() {}
      
      public Item(String itemName, Integer price, Integer quantity) { 
          this.itemName = itemName;
          this.price = price;
          this.quantity = quantity;
      }     
}
```
- 테스트 코드
```java
public class BeanValidationTest {

      @Test
      void beanValidation() {
ValidatorFactory factory = Validation.buildDefaultValidatorFactory(); Validator validator = factory.getValidator();
Item item = new Item(); item.setItemName(" "); //공백
item.setPrice(0); item.setQuantity(10000);
Set<ConstraintViolation<Item>> violations = validator.validate(item); for (ConstraintViolation<Item> violation : violations) {
System.out.println("violation=" + violation);
System.out.println("violation.message=" + violation.getMessage()); }
} }
```

### Bean Vlidation - 스프링 적용

##### 스프링과 Bean Vlidation
- 스프링 부트가 spring-boot-starter-validation 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합
- 스프링 부트는 자동으로 글로벌 Validator로 등록
- `@Validated` 는 스프링 전용 검증 애노테이션이고, `@Valid` 는 자바 표준 검증 애노테이션이다. 둘 중 아무거나 사용해도 동일하게 작동

##### 검증 순서
- @ModelAttribute 각각의 필드에 타입 변환 시도
  -  성공하면 다음으로
  - 실패하면 typeMismatch 로 FieldError 추가 
- Validator 적용

##### 바인딩과 Bean Validation
- 모델 객체에 바인딩 받는 값이 정상으로 들어와야 검증도 의미가 있음(BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않음)
- @ModelAttribute 각각의 필드 타입 변환시도 변환에 성공한 필드만 BeanValidation 적용
- ValidationItemController
```java
@Slf4j
@Controller
@RequestMapping("/validation/v3/items")
@RequiredArgsConstructor
public class ValidationItemControllerV3 {

    private final ItemRepository itemRepository;
    
    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll(); 
        model.addAttribute("items", items);
        return "validation/v3/items";
    }
    
    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId); 
        model.addAttribute("item", item);
        return "validation/v3/item";
    }
    
    @GetMapping("/add")
    public String addForm(Model model) {
        model.addAttribute("item", new Item());
        return "validation/v3/addForm";
    }

    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
        // item에 검증
        if (bindingResult.hasErrors()) { 
            log.info("errors={}", bindingResult);
            return "validation/v3/addForm";
        }

        Item savedItem = itemRepository.save(item); 
        redirectAttributes.addAttribute("itemId", savedItem.getId()); 
        redirectAttributes.addAttribute("status", true);
        return "redirect:/validation/v3/items/{itemId}";
    }
    
    @GetMapping("/{itemId}/edit")
    public String editForm(@PathVariable Long itemId, Model model) {
        Item item = itemRepository.findById(itemId); 
        model.addAttribute("item", item);
        return "validation/v3/editForm";
    }
    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
        itemRepository.update(itemId, item);
        return "redirect:/validation/v3/items/{itemId}";
    }
}
```

### Bean Validation - 에러 코드

##### Bean Validation의 에러 코드
- Bean Validation을 적용하고 bindingResult 에 등록된 검증 오류 코드는 오류 코드가 애노테이션 이름으로 등록
- Bean Validation이 기본으로 제공하는 오류 메시지는 수정 가능
- 예
  - @NotBlank -> NotBlank.item.itemName NotBlank.itemName NotBlank.java.lang.String NotBlank
  - @Range -> Range.item.price Range.price Range.java.lang.Integer Range

##### BeanValidation 메시지 찾는 순서
- 생성된 메시지 코드 순서대로 messageSource 에서 메시지 찾기
- 애노테이션의 message 속성 사용 -> @NotBlank(message = "공백! {0}") 
- 라이브러리가 제공하는 기본 값 사용 공백일 수 없음

### Bean Validation - 오브젝트 오류

##### ScriptAssert
- 오브젝트 관련 오류( ObjectError )는 어떻게 처리는 `@ScriptAssert()` 를 사용
- Item
```java
@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000")
public class Item {
//...
}
```
- Item의 메시지 코드
  - ScriptAssert.item 
  - ScriptAssert

##### 커스텀 오브젝트 오류 처리
- 브젝트 오류(글로벌 오류)의 경우 @ScriptAssert 을 억지로 사용하는 것 보다는 다음과 같이 오브젝트 오류 관련 부분만 직접 자바 코드로 작성하는 것을 권장
- ValicationController
```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    if (item.getPrice() != null && item.getQuantity() != null) { 
         int resultPrice = item.getPrice() * item.getQuantity(); 
         if (resultPrice < 10000) {
             bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
        }       
    }

    if (bindingResult.hasErrors()) { 
        log.info("errors={}", bindingResult); return "validation/v3/addForm";
    }

    //성공 로직
    Item savedItem = itemRepository.save(item); 
    redirectAttributes.addAttribute("itemId", savedItem.getId()); 
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v3/items/{itemId}";
}
```

### Bean Validation의 한계와 groups

#### Bean Validation의 한계
- 등록할 때 모델과 수정할 때 모델에 검증 조건이 다를 수가 있음
- 동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하는 방법 필요
- 해결 방법 2가지
  - BeanValidation의 groups 기능을 사용(권장되지 않는 기능)
  - Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어서 사용

##### BeanValidation groups 기능
- 등록시에 검증할 기능과 수정시에 검증할 기능을 각각 그룹으로 나누어 적용
- 저장용 groups
```java
public interface SaveCheck {
}
```
- 수정용 groups
```java
public interface UpdateCheck {
  }
```
- groups를 적용한 모델
```java
@Data
public class Item {
    @NotNull(groups = UpdateCheck.class) //수정시에만 적용 
    private Long id;
    
    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class}) 
    private String itemName;
    
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
    private Integer price;
    
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
    @Max(value = 9999, groups = SaveCheck.class) //등록시에만 적용 
    private Integer quantity;
    
    public Item() {}
    public Item(String itemName, Integer price, Integer quantity) { 
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    } 
}
```
- Groups를 적용한 컨트롤러
```java
@PostMapping("/add")
public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
}
```
```java
@PostMapping("/{itemId}/edit")
public String editV2(@PathVariable Long itemId, @Validated(UpdateCheck.class) @ModelAttribute Item item, BindingResult bindingResult) {
}
```

##### 객체 분리
- 폼 데이터 전달을 위한 별도의 객체 사용
  - 쉬운 방법: HTML Form -> Item -> Controller -> Item -> Repository
  - 추천 방법: HTML Form -> `ItemSaveForm` -> Controller -> `Item` 생성 -> Repository
- 네이밍
  - ItemSaveForm , ItemSaveRequest , ItemSaveDto 등으로 사용
- Item
```java
@Data
  public class Item {
      private Long id;
      private String itemName;
      private Integer price;
      private Integer quantity;
}
```
- ItemSaveForm
```java
@Data
public class ItemSaveForm {
    @NotBlank
    private String itemName;
    
    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;
    
    @NotNull
    @Max(value = 9999)
    private Integer quantity;
}
```
 - ItemUpdateForm
 ```java
 @Data
public class ItemUpdateForm {
    @NotNull
    private Long id;
    
    @NotBlank
    private String itemName;
    
    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;
    
    private Integer quantity;
}
 ```
- ValidationItemController
```java
@Slf4j
@Controller
@RequestMapping("/validation/v4/items")
@RequiredArgsConstructor
public class ValidationItemControllerV4 {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll(); 
        model.addAttribute("items", items);
        return "validation/v4/items";
    }

    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId); 
        model.addAttribute("item", item);
        return "validation/v4/item";
    }

    @GetMapping("/add")
    public String addForm(Model model) {
        model.addAttribute("item", new Item());
            return "validation/v4/addForm";
    }

    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute("item") ItemSaveForm form,
BindingResult bindingResult, RedirectAttributes redirectAttributes) {
    
    //특정 필드 예외가 아닌 전체 예외
    if (form.getPrice() != null && form.getQuantity() != null) {
        int resultPrice = form.getPrice() * form.getQuantity(); 
        if (resultPrice < 10000) {
            bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
        } 
    }
    
    if (bindingResult.hasErrors()) { 
        log.info("errors={}", bindingResult); 
        return "validation/v4/addForm";
    }

    //성공 로직(변환)
    Item item = new Item(); 
    item.setItemName(form.getItemName()); 
    item.setPrice(form.getPrice()); 
    item.setQuantity(form.getQuantity());
    Item savedItem = itemRepository.save(item); 
    redirectAttributes.addAttribute("itemId", savedItem.getId()); 
    redirectAttributes.addAttribute("status", true);
    return "redirect:/validation/v4/items/{itemId}";
}
    @GetMapping("/{itemId}/edit")
    public String editForm(@PathVariable Long itemId, Model model) {
        Item item = itemRepository.findById(itemId); 
        model.addAttribute("item", item);
        return "validation/v4/editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @Validated @ModelAttribute("item") ItemUpdateForm form, BindingResult bindingResult) {
        
        //특정 필드 예외가 아닌 전체 예외
        if (form.getPrice() != null && form.getQuantity() != null) {
            int resultPrice = form.getPrice() * form.getQuantity(); 
            if (resultPrice < 10000) {
                bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }            
        }
        
        if (bindingResult.hasErrors()) { 
            log.info("errors={}", bindingResult); return "validation/v4/editForm";
        }

        //성공 로직(변환)
        Item itemParam = new Item(); 
        itemParam.setItemName(form.getItemName()); 
        itemParam.setPrice(form.getPrice()); 
        itemParam.setQuantity(form.getQuantity());
        itemRepository.update(itemId, itemParam);
        return "redirect:/validation/v4/items/{itemId}";
      }
}
```

#### Bean Validation - HTTP 메시지 컨버터

##### @ModelAttribute와 @RequestBody
- @ModelAttribute 는 HTTP 요청 파라미터(URL 쿼리 스트링, POST Form)를 다룰 때 사용
- @RequestBody 는 HTTP Body의 데이터를 객체로 변환할 때 사용(`HttpMessageConverter`)

##### Bean Validation HTTP 메시지 컨버터에 적용하기
- @ModelAttribute와 @RequestBody 둘다 @Validated, @Valid로 검증 가능
- @ModelAttribute 는 필드 단위로 정교하게 바인딩이 적용되며 특정 필드가 바인딩 되지 않아도 나머지 필드는 정상 바인딩 되고, Validator를 사용한 검증도 적용할 수 있음
- @RequestBody 는 HttpMessageConverter 단계에서 JSON 데이터를 객체로 변경하지 못하면 이후 단계 자체가 진행되지 않고 예외가 발생
- ValidationItemApiController
```java
@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {

    @PostMapping("/add")
    public Object addItem(@RequestBody @Validated ItemSaveForm form, BindingResult bindingResult) {
        log.info("API 컨트롤러 호출");
         if (bindingResult.hasErrors()) {
             log.info("검증 오류 발생 errors={}", bindingResult);
             return bindingResult.getAllErrors(); 
        }
        log.info("성공 로직 실행");
        return form;
      }
}
```

##### API의 3가지 경우
- 성공 요청: 성공
- 실패 요청: JSON을 객체로 생성하는 것 자체가 실패함
- 검증 오류 요청: JSON을 객체로 생성하는 것은 성공했고, 검증에서 실패함

로그인 처리1
=======
### 로그인 처리1

##### 홈 화면
- HomeController
```java
@GetMapping("/")
public String home() {
    return "home";
}
```
- home.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org"> 
<head>
   <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}" href="../css/bootstrap.min.css" rel="stylesheet">
</head>

<body>
<div class="container" style="max-width: 600px"> <div class="py-5 text-center">
<h2>홈 화면</h2> </div>
    <div class="row">
        <div class="col">
              <button class="w-100 btn btn-secondary btn-lg" type="button" th:onclick="|location.href='@{/members/add}'|">회원 가입
              </button>
        </div>
        <div class="col">
            <button class="w-100 btn btn-dark btn-lg" onclick="location.href='items.html'" th:onclick="|location.href='@{/login}'|" type="button">로그인
              </button>
        </div>
    </div>
<hr class="my-4">
</div> <!-- /container -->
</body>
</html>
```

##### 회원 가입
- Member
```java
@Data
public class Member {
    private Long id;

    @NotEmpty
    private String loginId; //로그인 ID @NotEmpty
    private String name; //사용자 이름 @NotEmpty
    private String password;
}
```
- MemberRepository
```java
@Slf4j
@Repository
public class MemberRepository {

     private static Map<Long, Member> store = new HashMap<>(); //static 사용

    private static long sequence = 0L; //static 사용
    
    public Member save(Member member) { 
        member.setId(++sequence); 
        log.info("save: member={}", member); 
        store.put(member.getId(), member); 
        return member;
    }
    
    public Member findById(Long id) { 
        return store.get(id);
    }

    public Optional<Member> findByLoginId(String loginId) { 
        return findAll().stream().filter(m -> m.getLoginId().equals(loginId)) .findFirst();
    }

    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }
    
    public void clearStore() { 
        store.clear();
    } 
} 
```
- MemberController
```java
@Controller
@RequiredArgsConstructor
@RequestMapping("/members")
public class MemberController {

    private final MemberRepository memberRepository;
    
    @GetMapping("/add")
    public String addForm(@ModelAttribute("member") Member member) { 
        return "members/addMemberForm";
    }
    
    @PostMapping("/add")
    public String save(@Valid @ModelAttribute Member member, BindingResult result) {
        
        if (result.hasErrors()) {
            return "members/addMemberForm";
        }
        
        memberRepository.save(member);
              return "redirect:/";
      }
}
```
- addMemberForm.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org"> 
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
href="../css/bootstrap.min.css" rel="stylesheet"> 
<style>
.container { max-width: 560px;
}
.field-error {
border-color: #dc3545;
            color: #dc3545;
}
</style>
</head>

<body>
<div class="container">
    <div class="py-5 text-center"> <h2>회원 가입</h2>
    </div>
    
    <h4 class="mb-3">회원 정보 입력</h4>
    <form action="" th:action th:object="${member}" method="post">
        <div th:if="${#fields.hasGlobalErrors()}">
            <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">전체 오류 메시지</p> 
        </div>

        <div>
            <label for="loginId">로그인 ID</label>control"
            <input type="text" id="loginId" th:field="*{loginId}" class="form-control" th:errorclass="field-error">

            <div class="field-error" th:errors="*{loginId}" /> 
        </div>

        <div>
            <label for="password">비밀번호</label>
            <input type="password" id="password" th:field="*{password}" class="form-control" th:errorclass="field-error">
            <div class="field-error" th:errors="*{password}" />
        </div>

        <div>
           <label for="name">이름</label>
            <input type="text" id="name" th:field="*{name}" class="form-control" th:errorclass="field-error">
            <div class="field-error" th:errors="*{name}" />
        </div>

        <hr class="my-4">

        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">회원가입</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg" onclick="location.href='items.html'" th:onclick="|location.href='@{/}'|" type="button">취소</button>
            </div>
        </div>
    </form>
</div> <!-- /container --> 
</body>
</html>
```
##### 로그인 기능
- LoginService
```java
@Service
@RequiredArgsConstructor
public class LoginService {
    private final MemberRepository memberRepository;
    
    /**
    * @return null이면 로그인 실패
    */
    public Member login(String loginId, String password) { 
        return memberRepository.findByLoginId(loginId).filter(m -> m.getPassword()
        .equals(password)) 
        .orElse(null);
    }   
}
```
- LoginForm
```java
@Data
public class LoginForm {
    @NotEmpty
    private String loginId;
    @NotEmpty
    private String password;
}
```
- LoginController
```java
@Slf4j
@Controller
@RequiredArgsConstructor
public class LoginController {

    private final LoginService loginService;
    
    @GetMapping("/login")
    public String loginForm(@ModelAttribute("loginForm") LoginForm form) {
        return "login/loginForm";
    }
    
    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult) {
        
        if (bindingResult.hasErrors()) { 
            return "login/loginForm";
        }
        
        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

        log.info("login? {}", loginMember); 
         
        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        return "redirect:/";
    }
}
```
- logfinFomr.html
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org"> 
<head>
<meta charset="utf-8">
<link th:href="@{/css/bootstrap.min.css}" href="../css/bootstrap.min.css" rel="stylesheet"> 
<style>
.container { max-width: 560px;
}
.field-error {
border-color: #dc3545;
              color: #dc3545;
          }
</style>
</head>

<body>
    <div class="container">
        <div class="py-5 text-center">
            <h2>로그인</h2> </div>
            
            <form action="item.html" th:action th:object="${loginForm}" method="post">
                <div th:if="${#fields.hasGlobalErrors()}">
                    <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">전체 오류 메시지</p> </div>
                    <div> 
                        <label for="loginId">로그인 ID</label>
                        <input type="text" id="loginId" th:field="*{loginId}" class="form-control" th:errorclass="field-error">
                        <div class="field-error" th:errors="*{loginId}" />
                    </div> 
                    <div>
                        <label for="password">비밀번호</label>
                        <input type="password" id="password" th:field="*{password}" class="form-control" th:errorclass="field-error">
                        <div class="field-error" th:errors="*{password}" />
                    </div>
                    <hr class="my-4">
                    <div class="row">
                        <div class="col">
                            <button class="w-100 btn btn-primary btn-lg" type="submit">로그인</button> 
                        </div>
                        <div class="col">
                            <button class="w-100 btn btn-secondary btn-lg" onclick="location.href='items.html'" th:onclick="|location.href='@{/}'|" type="button">취소</button>
                        </div>
                    </div>
            </form>
    </div>
</body>
</html>
```

### 로그인 처리1 - 쿠기

##### 쿠키
- 서버에서 로그인에 성공하면 HTTP 응답에 Set-Cookie로 쿠키를 담아서 브라우저에 전달하면 `브라우저는 앞으로 해당 쿠키를 지속해서 보내줌`
- 서버에서 쿠키를 생성하고 HttpServletResponse에 담아서 클라이언트에서 전달 (쿠키 이름은 memberId 이고, 값은 회원의 id)
- 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지
- 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지

##### 쿠키를 사용한 로그인 처리
- 클라이언트
  - Cookie: memberId=1
- 서버의 쿠키 저장소
  - memberId: 1
- LoginController
```java
@PostMapping("/login")
public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
    
    if (bindingResult.hasErrors()) { 
        return "login/loginForm";
    }
    
    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
    log.info("login? {}", loginMember); 
    
    if (loginMember == null) {
        bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
        return "login/loginForm";
    }
    
    Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
    response.addCookie(idCookie); 
    return "redirect:/";
}
```
- HomeController
```java
@Slf4j
@Controller
@RequiredArgsConstructor
public class HomeController {

    private final MemberRepository memberRepository;


    @GetMapping("/")
    public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId,
  Model model) {
        if (memberId == null) {
              return "home";
        }
        
        Member loginMember = memberRepository.findById(memberId); 
        
        if (loginMember == null) {
              return "home";
        }
        
        model.addAttribute("member", loginMember);
        return "loginHome";
      }

    @PostMapping("/logout")
    public String logout(HttpServletResponse response) {
        expireCookie(response, "memberId");
        return "redirect:/";
    }
    
    private void expireCookie(HttpServletResponse response, String cookieName) { 
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
}
```
- loginHome.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org"> 
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}" href="../css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container" style="max-width: 600px"> 
    <div class="py-5 text-center">
        <h2>홈 화면</h2> </div>
        <h4 class="mb-3" th:text="|로그인: ${member.name}|">로그인 사용자 이름</h4>
        <hr class="my-4">
        <div class="row">
          <div class="col">
              <button class="w-100 btn btn-secondary btn-lg" type="button" th:onclick="|location.href='@{/items}'|">상품 관리
              </button>
          </div>
          <div class="col">
              <form th:action="@{/logout}" method="post"> <button class="w-100 btn btn-dark btn-lg" onclick="location.href='items.html'" type="submit"> 로그아웃 </button>
              </form>
            </div>
        </div>
        <hr class="my-4">
    </div> <!-- /container -->
</body>
</html>
```

##### 쿠키와 보안 문제
- 문제점
  - 쿠키 값은 임의로 변경 가능
  - 클라이언트의 쿠키에 보관된 정보가 해킹될 수 있음
  - 해커가 쿠키를 한번 훔쳐가면 평생 사용 가능
- 대안
  - 쿠키에 중요한 값을 노출하지 않고 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고 서버에서 토큰과 사용자 id를 매핑해서 인식
  - 토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능해야함
  - 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게 유지

### 로그인 처리1 - 세션

##### 세션
- 쿠키에 중요한 정보를 보관하는 방법은 여러가지 보안 이슈가 있어서 중요한 정보는 모두 서버에 저장하고 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결하는 방식
- 데이터 전달 자체는 쿠키를 사용
-  서버는 로그인 인증 성공 시에 세션 ID를 생성하는데, `추정 불가능한 UUID로 토큰 생성`하여 서버 저장소에 식별한 값과 사용자 정보를 매핑하여 저장

##### 세선과 보아 문제
- 쿠키 값을 변조 -> 예상 불가능한 UUID로 토큰을 생성하여 방어 가능
- 클라이언트의 쿠키에 보관된 정보가 해킹될 수 있음 -> 세션 ID가 해킹을 당해도 중요한 정보는 다 서버의 세션 저장소에 있으므로 방어 가능
- 해커가 쿠키를 한번 훔쳐가면 평생 사용 가능 -> 서버에서세션의 만료시간을 짧게 유지

##### 세션를 사용한 로그인 처리
- 클라이언트
  - Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61
- 서버의 세션 저장소
  - sessionId: zz0101xx-bab9-4b92-9b32-dadb280f4b61
  - value: 회원 정보


##### 세션를 사용한 로그인 처리 HttpSession
- SessionConst
```java
public class SessionConst {
    public static final String LOGIN_MEMBER = "loginMember";
}
```
- LoginController
```java
@PostMapping("/login")
public String loginV3(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request) {
    
    if (bindingResult.hasErrors()) { 
        return "login/loginForm";
    }
    
    Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
    log.info("login? {}", loginMember); 
    
    if (loginMember == null) {
        bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
        return "login/loginForm";
    }

    //로그인 성공 처리
    //세션이 있으면 있는 세션 반환, 없으면 신규 세션 생성
    HttpSession session = request.getSession(); //세션에 로그인 회원 정보 보관
    session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

    return "redirect:/";
}

@PostMapping("/logout")
public String logoutV3(HttpServletRequest request) {
    
    //세션을 삭제
    HttpSession session = request.getSession(false); 
    
    if (session != null) {
        session.invalidate(); 
    }
    
    return "redirect:/";
}
```
- HomeController - @SessionAttribute를 사용하지 않은 방법
```java
@GetMapping("/")
public String homeLoginV3(HttpServletRequest request, Model model) {
    
    //세션이 없으면 home
    HttpSession session = request.getSession(false); 
    
    if (session == null) {
        return "home";
    }
    
    Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
    //세션에 회원 데이터가 없으면 home 
    
    if (loginMember == null) {
        return "home";
    }
    
    //세션이 유지되면 로그인으로 이동 
    model.addAttribute("member", loginMember); 
    return "loginHome";
}
```
- HomeController - @SessionAttribute를 사용한 방법
```java
@GetMapping("/")
public String homeLoginV3Spring(@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model) {
    
    //세션에 회원 데이터가 없으면 home 
    if (loginMember == null) {
          return "home";
    }
      
    //세션이 유지되면 로그인으로 이동 
    model.addAttribute("member", loginMember); 
    return "loginHome";
}
```

##### 세션 정보
- 세션 정보
  - sessionId: 세션Id, JSESSIONID 의 값
  - maxInactiveInterval: 세션의 유효 시간
  - creationTime: 세션 생성일시
  - lastAccessedTime: 세션과 연결된 사용자가 최근에 서버에 접근한 시간, 클라이언트에서 서버로 sessionId(JSESSIONID)를 요청한 경우에 갱신
  - isNew: 새로 생성된 세션인지 아니면 이미 과거에 만들어졌고, 클라이언트에서 서버로 sessionId (JSESSIONID)를 요청해서 조회된 세션인지 여부
- SessionInfoController
```java
@Slf4j
@RestController
public class SessionInfoController {

     @GetMapping("/session-info")
     public String sessionInfo(HttpServletRequest request) {
         HttpSession session = request.getSession(false); 
         
         if (session == null) {
             return "세션이 없습니다."; 
        }
        
        //세션 데이터 출력 
        session.getAttributeNames().asIterator()
        .forEachRemaining(name -> log.info("session name={}, value={}", name, session.getAttribute(name)));
        
        log.info("sessionId={}", session.getId()); 
        log.info("maxInactiveInterval={}", session.getMaxInactiveInterval()); 
        log.info("creationTime={}", new Date(session.getCreationTime())); 
        log.info("lastAccessedTime={}", new Date(session.getLastAccessedTime())); 
        log.info("isNew={}", session.isNew());
        return "세션 출력"; 
    }
}
```
##### 타임 아웃 설정
- HttpSession은 세션 생성 시점이 아니라 사용자가 서버에 최근에 요청한 시간을 기준으로 타임 아웃이 동작
- 세션 삭제
  - session.invalidate() 가 호출 되는 경우에 삭제됨
- 세션 타임 설정
  - 프로퍼티 값 수정을 통한 글로벌 세션 타임 아웃 설정: `server.servlet.session.timeout=60`
  - 코드에서 HttpSession 수정을 통한 세션 별 타임 아웃 설정: `session.setMaxInactiveInterval(1800);`


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
