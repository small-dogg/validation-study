# Validation

## V1 : addAtrribute Map
Model에 입련된 값의 문제점을 Map 형태로 담아, 화면에서 추가 처리

```java
Map<String, String> errors = new HashMap<>();
errors.put("price","가격은 1000원 이상이어야 합니다.");
```
```html
<input type="text" th:classappend="${errors?.containsKey('itemName')} ? 'fielderror' : _"
       class="form-control">

<input type="text" class="form-control field-error">

<div class="field-error" th:if="${errors?.containsKey('itemName')}" th:text="$
{errors['itemName']}">
    상품명 오류
</div>
```

## V2 : BindingResult
BindingResult 객체는 V1에서 처리했던 Error처리를 보다 쉽게 도와주는  스프링에서 제공하는 객체이다.

> BindingResult Argument는 반드시 ModelAttribute Argument 뒤에 위치해야한다.
> 
> public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, ...)

BindingResult는 ObjectError 객체를 담아 이를 View에 반환하는데, 특정 필드가 명확할 경우, FieldError를 사용한다.
예시에서 Item 객체의 필드에 오류가 존재할 경우, 
```java
bindingResult.addError(new FieldError("item", "itemName", "이름을 입력하세요"));
```
와 같은 형태로 입력할 수 있으며,

특정 필드가 존재하지 않을 경우 ObjectError 객체를 생성하여 Global Error를 담으면 된다.
```java
bindResult.addError(new ObjectError("item","가격 * 수량의 합은 10,000원 이상이어야합니다."));
```

Thymeleaf 에서는 bindingResult에 대해 ${#fields.xxx()}로 접근할 수 있다.
```html
<div th:if="${#field.hasGlobalErrors()}">
    <p class=""field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}"></p>
</div>
<div>
    <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
    <input type="text" id="itemName" th:field="*{itemName}"
           th:errorclass="field-error" class="form-control"
           placeholder="이름을 입력하세요">
    <div class="field-error" th:errors="*{itemName}">
        상품명 오류
    </div>
</div>
```

## BindingResult 메시지 적용

## BindingResult rejectValue 사용
rejectValue, reject : MessageCodesResolver 

(범용성이 좋지만, 세밀하게 작업할 수 없음) tradeoff
required vs required.item.itemName

메시지를 범용성있게 작성하기 시작해서, 세밀하게 작성해야하는 경우 차근차근 점진적으로 세밀하게 작성.

[v1] required : 필수 값입니다.

[v2] required.item.itemName : 항목 명은 필수 값 입니다.


 new String{"required.item.itemName,required"}
 
위의 형태를 스프링에서는 MessageCodesResolver로 구현해둠.

# Bean Validation

## 하비어네이트 Validator 참고 자료
- [공식 사이트](http://hibernate.org/validator/)
- [공식 메뉴얼](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/)
- [검증 애노테이션 모음](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)

## Bean Validation

### 사용 방법
@Validated 사용 방법

### Bean Validation의 한계
groups - 실무에서 잘 사용하지 않음

### Form 전송 객체
요청별로 DTO 만들어서 사용

### HTTP 메시지 컨버터
동일하게 사용할 수있지만, 타입미스매치를 처리하기 어려움
요청받은 정보를 객체화해야하지만, 객체화할 수 없어 에러 발생
-> 예외처리로 해결해야함.