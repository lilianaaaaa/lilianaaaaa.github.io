# JAVA 8 Syntax

### Optional

- 어떤 데이터가 null이 올 수 있는 경우에는 해당 값을 Optional로 감싸서 생성할 수 있다.
- Optional은 null 또는 실제 값을 value로 갖는 wrapper로 감싸서 NPE(NullPointerException)로부터 자유로워지기 위해 나온 Wrapper 클래스이다.
- 따라서 Optional을 반환하는 메소드는 절대 null을 갖는 value를 반환해서는 안된다.
- 또한 Optional은 값을 Wrapping하고 다시 풀고, null일 경우에는 대체하는 함수를 호출하는 등의 오버헤드가 있으므로 성능이 저하될 수 있다.
- 그렇기 때문에 메소드의 반환 값이 절대 null이 아니라면 Optional을 사용하지 않는 것이 성능저하가 적다.
- 즉, Optional은 메소드의 결과가 null이 될 수 있으며, 클라이언트가 이 상황을 처리해야 할 때 사용하는 것이 좋다.
- 생성하기

```java
//빈 객체 생성
Optional<String> optional = Optional.empty();
//optional.isPresent로 null 체크

// Optional의 value는 값이 있을 수도 있고 null 일 수도 있다. 
Optional<String> optional = Optional.ofNullable(getName()); 
String name = optional.orElse("anonymous"); // 값이 없다면 "anonymous" 를 리턴
```

- 사용하기
    - 기존에는 아래와 같이 null 검사를 한 후에 null일 경우에는 새로운 객체를 생성해주어야 했다. 이러한 과정을 코드로 나타내면 다소 번잡해보이는데, Optional<T>와 Lambda를 이용하면 해당 과정을 보다 간단하게 표현할 수 있다.

```java
// Java8 이전 
List<String> names = getNames(); 
List<String> tempNames = list != null ? list : new ArrayList<>(); 

//Optional<T>와 lambda 이용
List<String> nameList = Optional.ofNullable(getList()).orElseGet(() -> new ArrayList<>());
```

- 예시 코드

```java
//Null 검사 코드
public String findPostCode() { 
	UserVO userVO = getUser(); 
	if (userVO != null) { 
		Address address = user.getAddress(); 
		if (address != null) { 
			String postCode = address.getPostCode(); 
			if (postCode != null) { 
				return postCode; 
			} 
		} 
	} return "우편번호 없음"; 
}

//Optional 사용 코드
public String findPostCode() { 
// 위의 코드를 Optional로 펼쳐놓으면 아래와 같다. 
	Optional<UserVO> userVO = Optional.ofNullable(getUser()); 
	Optional<Address> address = userVO.map(UserVO::getAddress); 
	Optional<String> postCode = address.map(Address::getPostCode); 
	String result = postCode.orElse("우편번호 없음"); 
	
	// 그리고 위의 코드를 다음과 같이 축약해서 쓸 수 있다. 
	String result = user.map(UserVO::getAddress) 
		.map(Address::getPostCode) 
		.orElse("우편번호 없음"); 
}
```

- 프로젝트 활용 예시 코드

```java
@Override
public UserDetails loadUserByUsername(String userId) throws UsernameNotFoundException {
    Optional<UserEntity> userEntityWrapper = userRepository.findOne(QUserEntity.USER.userId.lower().eq(userId.toLowerCase()));

    if (!userEntityWrapper.isPresent())
        throw new UsernameNotFoundException(userId + " - 해당하는 유저를 찾을 수 없습니다.");

    UserEntity userEntity = userEntityWrapper.get();

    return new CustomUser(
            userEntity.getUserId(),
            userEntity.getPassword(),
            Arrays.asList(new SimpleGrantedAuthority("ROLE_".concat(userEntity.getGradeEntity().getName()))),
            userEntity.getNo(),
            userEntity.getUserName(),
            userEntity.getSecessionAt().equals("N") ? true : false,
            userEntity.getConfirmAt().equals("C") ? true : false);
}
```

- Optional의  **orElse VS orElseGet 사용 정리**
    - orElse
        - Optional의 값이 null이든 아니든 항상 호출된다.
        - 그에 따른 비용이 추가되고, 문제가 발생할 수 있다.
        - 값이 미리 존재하는 경우에 사용한다.
    - orElseGet
        - Optional의 값이 null일 경우에만 호출된다.
        - 비용이 orElse보다 저렴하며, 불필요한 문제가 발생하지 않는다.
        - 값이 미리 존재하지 않는 거의 대부분의 경우에 orElseGet을 사용하면 된다.

출처:
[https://mangkyu.tistory.com/70](https://mangkyu.tistory.com/70)




# Annotation

### @Valid @Validated

- Spring 데이터 검증시 사용
- 객체에 들어가는 값을 검증해주는 어노테이션
- @Min, @Max, @NotNull 등의 어노테이션으로 간단하게 값을 체크할 수 있다.
- validation-api가 아닌 hibernate-validator를 추가해야 함
    - validation-api 라이브러리를 사용하면 @Range, @NotNull, @Size 등의 어노테이션으로 검증 불가
- 에러 체크는 hasErrors 함수 사용
- 에러 리스트는 getAllErrors() 함수로 가져온다
- getDefaultMessage()함수로 defaultError 메세지를 가져올 수 있다.
- 동작 과정
    - 빈 생성
        - **javax.validation.Validator**
        - **org.springframework.validation.beanvalidation.OptionalValidatorFactoryBean**
    - RequestMappingHandlerAdapter 어뎁터 안에서 Controller를 doInvoke하기전에 getMethodArgumentValues 안에서 핸들러 메서드를 위한 여러가지의 작업을 진행해 주는데 그 중 Validate작업이 여기에 포함
    - 해당 부분은 ModelAttributeMethodProcessor (HandlerMethodArgumentResolver)를 사용하게 되며 해당 클레스의 resolveArgument 에서는 모델의 유효성 검사 또는 바인딩 작업등을 진행해주게 됩니다.
- 사용 방법
    - dependency 추가
    - **VO에 어노테이션을 valid 설정을 한다.**
    
    ```java
    @NotNull @NotEmpty    private String userId;              // 사용자 아이디
    
    @Pattern(regexp = "^[_0-9a-zA-Z-]+@[0-9a-zA-Z]+(.[_0-9a-zA-Z-]+)*$", message = "not email")
    @Size(min = 4, max = 45, message = "name is 4 to 45")
    @Max(value = 3, message = "not allow type")
    @Min(value = 1, message = "not allow type")
     
    @AssertFalse  거짓인지?
    @AssertTrue 참인지?
    @DecimalMax 지정 값 이하의 실수인지?
    @DecimalMin 지정 값 이상의 실수인지?
    @Digits(integer=,fraction=) 정수 여부
    @Future 미래 날짜인지?
    @Max 지정 값 이상인지?
    @Min 지정 값 이하인지?
    @NotNull  Null이 아닌지?
    @Null Null인지?
    @Pattern(regex=,flag=) 정규식을 만족하는지?
    @Past   과거날짜인지?
    @Size(min=,max=)  문자열 또는 배열등의 길이 만족 여부
    ```
    
    - Controller
    
    ```java
    // 사용자 추가
    @ResponseBody
    @PostMapping("/join")
    public ResponseEntity join(@ModelAttribute @Valid AccountDto accountDto, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) return new ResponseEntity(HttpStatus.BAD_REQUEST);
        try {
            accountDto.setConfirmAt("C");
            userService.joinUser(accountDto);
            return new ResponseEntity(HttpStatus.OK);
        } catch (Exception e) {
            e.printStackTrace();
            return new ResponseEntity(HttpStatus.BAD_REQUEST);
        }
    }
    ```
