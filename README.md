# about_Spring_Boot
스프링 부트를 사용하며 알게된 것을 기록해두는 저장소입니다.
<br/><br/><br/>



# application.properties
- @Value를 통해 자바 코드에서 값 사용 가능.
<br/><br/><br/>



# 어노테이션 

## @RestController 
- Restful Web API를 좀 더 쉽게 만들기 위한 기능.
- @Controller와 @ResponseBody를 합쳐놓은 어노테이션. 

## @Controller
- 해당 클래스를 요청을 처리해주는 컨트롤러로 사용시킴

## @ResponseBody
- 자바 객체를 HTTP 응답 본문의 객체로 변환해 클라이언트에게 전송.


## @GetMapping
- 클라이언트의 요청을 처리할 URL을 매핑

## Lombok 자주 사용 어노테이션들
```
cf) gradle implementation 'org.projectlombok:lombok'
		    annotationProcessor 'org.projectlombok:lombok' 설정.
@Getter/Setter
@ToString
@ToSring(exclude={"변수명"})
@NonNull
@EqualsAndHashCode
@Builder
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
@Log
@Value
@Data
```
<br/><br/><br/>



# DTO(Data Transfer Object)
- **데이터를 주고 받을 때는 Entity 클래스 자체를 반환하면 안되고 데이터 전달용 객체(DTO)를 사용해야함 !!!**
- 디비 설계를 외부에 노출할 필요도 없으며, 요청과 응답 객체가 항상 엔티티와 같지 않기 떄문.
<br/><br/><br/>



# JPA
## 영속성 컨텍스트
- 엔티티를 영구 저장하는 환경으로 엔티티 매니저를 통해 영속성 컨텍스트에 접근함. 
- 애플리케이션과 데이터베이스 사이의 중간계층 -> 버퍼링 캐싱 등의 장점
- 1차 캐시 : Map구조. entityManager.find() 호출 시 1차캐시 조회
- 동일성 보장 : 하나의 트랜잭션에서 같은 키값으로 영속성 컨텍스트에 저장된 엔티티 조회 시 같은 엔티티 조회를 보장. 바로 1차캐시에 저장된 엔티티 조회하기 때문. 
- 트랜잭션 지원하는 쓰기 지연 : sql을 쌓아두고 커밋시점에 sql 문들이 flush. 성능 향상
- 변경 감지. : JPA 1차 캐시에서 데이터베이스 불러온 엔티티 스냅샷을 갖고 있음. 스냅샷 변경 내용 있으면 update문을 쓰기 지연 저장소에 담아두고 커밋시점에 반영. 
```java
//예시
Item item = new Item(); // 영속성 컨텍스트에 저장할 엔티티 생성
item.setItemNm("테스트 상품"); /
EntityManager em = entityManagerFactory.createEntityManager(); //엔티티 매니저 팩토리로부터 엔티티 매니저 생성
EntityTransaction transaction = em.getTransaction(); // 엔티티 매니저 데이터 변경 시 데이터 무결성을 위해 반드시 트랜잭션 시작해야함. 
transaction.begin();
em.persiste(item); // 영속성 컨텍스트에 저장된상태. 아직 sql문을 보내지 않은 단계.
transaction.commit(); // 트랜잭션을 데이터베이스에 반영. 이때 영속성 컨텍스트에 저장된상품 정보가 데이터베이스 insert되면서 반영됨.
em.close(); //자원 반환
emf.close();
```
## 엔티티 생명 주기
```
비영속 - new 키워드를 통해 생성된 상태로 영속성 컨텍스트와 관련 없는 상태
영속 - 엔티티가 영속성 컨텍스트에 저장된 상태로 영속성 컨텍스트에 의해 관리되는 상태
	- 영속 상태에서 데이터베이스에 저장되지 않으며, 트랜잭션 커밋 시점에 데이터베이스에 반영
준영속 상태 - 영속성 컨텍스트에 엔티티가 저장되었다가 분리된 상태
삭제 상태 - 영속성 컨텍스트와 데이터베이스에서 삭제된 상태
```

## 엔티티 매니저 팩토리
- 엔티티 매니저 인스턴스 관리하는 주체. 
- 애플리케이션 실행 시 한 개만 만들어짐.
- 사용자로부터 요청이 오면 엔티티 매니저 팩토리로부터 엔티티 매니저 생성
## 엔티티 매니저
- 영속성 컨텍스트에 접근하여 엔티티에 대한 데이터베이스 작업 제공.
- 내부적으로 데이터베이스 커넥션 사용.
```
// 엔티티 매니저 메소드
find() //영속성 컨텍스트에 엔티티 검색하고 없을 경우 데이터베이스에서 데이터 찾아 영속성 컨텍스트에 저장.
persist() //엔티티를 영속성 컨텍스트에 저장.
remove()
flush() //JPA는 영속성 컨텍스트에 데이터를 저장 후 트랜잭션이 끝날 때 flush()를 호출하여 데이터베이스에 반영
clear() // JPA는 영속성 컨텍스트로부터 엔티티를 조회후 영속성 컨텍스트에 엔티티가 없을 경우 디비 조회. 
```


## 엔티티 연관 관계 매핑
- 데이터베이스에서 테이블끼리 외래키를 통해 연관 관계를 맺듯, 엔티티끼리 연관 관계를 매핑해서 사용.
### 엔티티 연관 관계 매핑 종류
```종류
// 즉시로딩 : 엔티티 조회할 때 해당 엔티티와 매핑된 엔티티도 한 번에 조회
@OneToOne // 일대일 => 즉시 로딩이 기본 fetch전략 (fetch = FetchType.EAGER)
@OneToMany // 일대다
@ManyToOne // 다대일 => 즉시 로딩이 기본 fetch전략
@ManyToMany // 다대다
```
```java
//예시
@OneToOne //Member 엔티티와 일대일 매핑
@JoinColumn(name="member_id") //외래키 지정(name명시안할경우 JPA가 알아서 id를 찾지만 컬럼명이 원하는대로 생성되지 않을 수 있음.)
private Member member;
```

### 엔티티 연관 관계 매핑 방향
```방향
- 단방향
- 양방향
```
### 엔티티 연관 관계 주인
- 엔티티와 테이블은 다르기 때문에, 엔티티를 양방향 연관 관계로 설정하면 객체의 참조는 둘인데 외래키는 하나가 되는 상황이 됨.
- 둘 중 누가 외래키를 관리할 지 정해야함.
- 무조건 양방향으로 연관 관계를 매핑하면 해당 엔티티는 엄청나게 많은 테이블과 연관관계를 맺게되고 엔티티 클래스 자체가 복잡해지기 때문에, 단방향 매핑 후 나중에 필요할 경우 추가.
```주인
- 연관 관계 주인은 외래키가 있는 곳으로 설정
- 연관 관계의 주인이 외래키를 관리(등록, 수정, 삭제)
- 주인이 아닌 쪽은 연관 관계 매핑 시 mappedBy 속성의 값으로 연관 관계의 주인을 설정
- 주인이 아닌 쪽은 읽기만 가능
```
```java
//예시
@OneToMany(mappedBy = "order")
private List<OrderItem> orderItems = new ArrayList<>();
// 외래키가 orderItem 테이블에 있는 경우 => 연관 관계의 주인은 OrderItem 엔티티
// 주인이 아닌 곳에서 mappedBy로 주인 설정. 값은 주인의 외래키 필드 값으로 세팅.
```
#### 다대다 매핑
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다를 표현할 수 없음.
- 연결 테이블을 생성해서 다대다 관계를 일대다, 다대일 관계로 풀어냄. 
- 객체는 컬렉션을 사용해서 다대다 표현 가능.(리스트 형태)


### 영속성 전이
- 엔티티의 상태를 변경할 때 해당 엔티티와 연관된 엔티티의 상태 변화를 전파시키는 옵션.
- 부모는 One, 자식은 Many
```종류
PERSIST
MERGE
REMOVE
REFRESH
DETACH
ALL
```
### 고아 객체 제거
- 부모 엔티티와 연관 관계가 끊어진 자식 엔티티를 고아 객체
- 객체 제거 기능은 참조하는 곳이 하나일 때만 사용. 
- @OneToOne, @OneToMany 일 때 사용.
- orphanRemoval = true


### 지연로딩
- 즉시 로딩은 실무에서 사용하기 힘듬.
- fetch = FetchType.LAZY
- 지연 로딩으로 설정하면 실제 엔티티 대신에 프록시 객체를 넣어둠
- 프록시 객체는 실제로 사용되기 전까지 데이터 로딩을 하지 않고, 실제 사용 시점에 조회 쿼리문이 실행됨.



## JPA Auditing
- JPA에서는 엔티티가 저장 또는 수정될 때 자동으로 등록일, 수정일, 등록자, 수정자를 입력시켜줌.
- 이런 공통 멤버 변수들을 추상 클래스로 만들고, 해당 추상 클래스를 상속 받는 형태로 엔티티를 작성.
- @Configuration 빈에서 AuditorAware(i) 구현한 클래스를 사용하기




## JPAQuery(JPQL) 
### 쿼리 메소드
- find + (엔티티 이름(생략가능)) + By + 변수이름
- 조건이 많아질 때 쿼리 메소드를 선언하면 이름이 정말 길어지기도 함. 복잡한 쿼리를 다루기엔 부적합.
### @Query
- 쿼리메소드로 처리하기 힘든 복잡한 쿼리를 다룰 때 JPQL 사용
- JPQL은 엔티티 객체를 대상으로 쿼리를 수행. 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리. 
- JPQL은 SQL을 추상화해서 사용하기 때문에 특정 데이터베이스 SQL에 의존하지 않는다. 
- 단점 : @Query 어노테이션 안에 JPQL문법으로 문자열을 입력하기 때문에 잘못 입력하면 컴파일 시점에 에러를 발견할 수 없음. -> Querydsl 사용

### Querydsl
- Querydsl은 JPQL 빌더 오픈소스 프레임워크.
```
장점
1. 동적으로 쿼리 생성
2. 쿼리 재사용, 제약 조건 조립 및 가독성 향상
3. 컴파일 시점 오류 발견
4. 자동완성 생산성 향상.
```
- querydsl-apt => 엔티티를 기반으로 Q가 붙는 클래스 자동생성 해주는 플러그인
- 엔티티 위치가 변경되거나 삭제될 경우 기존 쿼리 타입(Q클래스)를 삭제해주어야 한다!
- 참조)https://gaemi606.tistory.com/entry/Spring-Boot-Querydsl-%EC%B6%94%EA%B0%80-Gradle-7x

### JPAQuery 메소드
```
//JPAQuery 데이터 반환 메소드
List<T> fetch() //조회 결과 리스트 반환
T fetchOne	//조회 대상이 1건인 경우 제네릭으로 지정한 타입 반환
T fetchFirst()	//조회 대상 중 1건만 반환
Long fetchCount()	//조회 대상 개수 반환
QueryResult<T> fetchResults()	//조회한 리스트와 전체 개수를 포함한 QueryResults 반환
```

#### QuerydslPredicateExcutor
- Predicate란 '이 조건이 맞다'라고 판단하는 근거를 함수로 제공하는 것.
```
// QuerydslPredicateExcutor 인터페이스 추상 메소드
long count(Predicate)
boolean exists(Predicate)
Iterable findAll(Predicate)
Page<T> findAll(Predicate, Pageable) // 조건에 맞는 페이지 데이터 반환. 
Iterable findAll(Predicate, Sort)
T findOne(Predicate)
``` 
- BooleanBuilder는 쿼리에 들어갈 조건을 만들어주는 빌더. Predicate를 구현하고 있으며 메소드 체인형식으로 사용 가능. 
<br/><br/><br/>




# Repository
- Data Access Object(DAO) 역할을 하는 인터페이스
```java
// JpaRepository<엔티티 클래스, 엔티티의 기본키 타입>
public interface ItemRepository extends JpaRepository<Item, Long>{}
```
- 기본적인 CURD 및 페이징 처리를 위한 메소드가 정의돼 있음.
```
// 주로사용하는 JpaRepository 메소드
<S extends T> save(S entity)
void delete (T entity)
count()
Iterable<T> findAll()
```
- Spring Data JPA 는 인터페이스만 작성하면 런타임 시점에서 자바의 Dynamic Proxy를 이용해서 객체를 동적으로 생성해줌. 따로 DAO와 xml 파일에 쿼리문 작성하지 않아도 됨.
<br/><br/><br/>



# 스프링 시큐리티 

## AuthenticationManager
- 스프링 시큐리티에서 인증은 AuthenticationManager을 통해 이루어지며, AuthenticationManagerBuilder가 AuthenticationManager를 생성함. userDetailService를 구현하고 있는 객체를 지정해주고, 비밀번호 암호화를 위해 passwordEncoder를 지정해주면 됨.
```java
protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        auth.userDetailsService(memberService).passwordEncoder(passwordEncoder());
    }
``` 

## CSRF(Cross Site Request Forgery)
```java
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}">
```
- 스프링 시큐리티를 사용할 경우 기본적으로 CSRF를 방어하기 위해 모든 POST 방식의 데이터 전송에는 CSRF 토큰 값이 있어야함. CSRF토큰은 실제 서버에서 허용한 요청이 맞는지 확인하기 위한 토큰. 사용자의 세션에 임의의 값을 저장하여 요청마다 그 값을 포함하여 전송하면 서버에서 세션에 저장되 값과 요청이 온 값이 일치하는지 확인하여 CSRF를 방어 


## UserDetailService (i)
- 이것을 구현하고 있는 클래스를 통해 로그인 기능 개발.
- UserDetailService인터페이스는 회원 정보를 가져오는 역할
- loadUserByUsername() 회원정보를 조회하여 사용자의 정보와 권한을 갖는 UserDetails 인터페이스 반환

## UserDetails (i)
- 회원정보를 담기 위해 사용하는 인터페이스 
- 직접 구현하거나 스프링 시큐리티에서 제공하는 User 클래스 사용. 


## spring-boot-validation
- 유효한 값인지 판단하는 소스가 여러 군데 흩어지면 관리하기 힘듬. 자바 빈 밸리데이션을 이용하면 객체의 값을 효율적으로 검증 가능.
``` 주요어노테이션
@NotEmpty
@NotBlank
@Length
@Email
@Max
@Min
@Null
@NotNull
```

```java
public String memberForm(@Valid MemberFormDto memberFormDto, BindingResult bindingResult){
	//검증하려는 객체의 앞에 @Valid 어노테이션을 선언하고, 파라미터로 bindingResult 객체를 추가. 검사 후 결과는 bindingResult에 담아줌. bindingResult.hasError()를 호출하여 에러가 있다면 회원가입 페이지로 이동. 
}
```


## spring-security-test
### MockMvc 테스트
- @AutoConfigureMockMvc 어노테이션 선언
- MockMvc 클래스를 이용해 실제 객체와 비슷하지만 테스트에 필요한 기능만 가지는 가짜 객체. MockMvc객체를 이용하면 웹브라우저에서 요청을 하는 것처럼 테스트할 수 있음.


## WebSecurityConfigurerAdapter
- WebSecurityConfigurerAdapter를 상속받는 클래스에 @EnableWebSecurity 어노테이션을 선언하면 SpringSecurityFilterChain이 자동으로 포함됨. WebSecutiryConfigureAdapter를 상속받아서 메소드 오버라이딩을 통해 보안 설정을 커스터마이징 가능

## HttpSecurity 
- http 요청에 대한 보안 설정.
- 페이지 권한 설정. 로그인 페이지 설정. 로그아웃 메소드 등에 대한 설정을 작성.

## BCryptPasswordEncoder
- 비밀번호를 위한 해시 메소드를 갖고 있음.

# Service Layer
## @Transactional
- 비즈니스 로직을 담당하는 서비스 계층 클래스에 @Transactional 어노테이션을 선언합니다. 로직을 처리하다가 에러가 발생하였다면, 변경된 데이터를 로직을 수행하기 이전의 상태로 콜백시켜줍니다. 
## @RequireArgsConstructor
- 빈을 주입하는 방법은 @Autowired 어노테이션 이용하거나, 필드 주입, 생성자 주입 이용이 있는데,
- @RequireArgsConstructor은 final이나 @NonNull이 붙은 필드에 생성자를 생성해줌. 
- 빈에 생성자가 1개이고 생성자의 파라미터 타입이 빈으로 등록이 가능하다면 @Autowired 어노테이션 없이 의존성 주입이 가능. 
<br/><br/><br/>
