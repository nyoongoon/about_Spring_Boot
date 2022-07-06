# about_Spring_Boot
스프링 부트를 사용하며 알게된 것을 기록해두는 저장소입니다.
<br/><br/><br/>



# application.properties
- @Value를 통해 자바 코드에서 값 사용 가능.
<br/><br/><br/>



# 어노테이션 

## @Controller
- 해당 클래스를 요청을 처리해주는 컨트롤러로 사용시킴

## @GetMapping
- 클라이언트의 요청을 처리할 URL을 매핑

## @PatchMapping
- HTTP메소드에서 PATCH는 요청된 자원의 일부만 업데이트할 때 PATCH 사용. 

## @RestController 
- Restful Web API를 좀 더 쉽게 만들기 위한 기능.
- @Controller와 @ResponseBody를 합쳐놓은 어노테이션. 

## @ResponseBody
- 자바 객체를 HTTP 응답 본문의 객체로 변환해 클라이언트에게 전송.

## @Transactional
- @Transactional(readOnly = true) -> 상품 데이터를 읽어오는 트랜잭션을 읽기 전용으로 설정. JPA가 더티체킹(변경감지)하지 않아 조회 성능 높힘.


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

# Async(비동기)
- 상품 주문에서 웹페이지의 새로고침 없이 서버에 주문을 요청하기 위함.

```java
//비동기 예시
public @ResponseBody ResponseEntity order
(@RequestBody @Valid OrderDto orderDto){
...
    return new ResponseEntity<Long>(orderId, HttpStatus.OK);   
}
```
- 스프링에서 비동기 처리를 할 때 @RequestBody와 @ResponseBody 어노테이션을 사용함.
- @Requestbody : HTTP 요청의 본문 body에 담긴 내용을 자바 객체로 전달.
- @ResponseBody : 자바 객체를 HTTP 요청의 body로 전달
- 결과값으로 생성된 주문번호와 요청이 성공했다는 HTTP 응답상태 코드 반환.
- 
### 비동기 프론트쪽 코드
- 스프링 시큐리티를 사용할 경우 기본적으로 POST 방식의 데이터 전송에는 CSRF 토큰 값이 필요하므로 해당 값들을 조회.
```javascript
function order(){
      var token = $("meta[name='_csrf']").attr("content"); //토큰 값 설정
      var header = $("meta[name='_csrf_header']").attr("content");

      // ... param

      $.ajax({
        url : url,
        type : "POST",
        contentType : "application/json",
        data : param,
        beforeSend : function (xhr){
          /* 데이터를 전송하기 전에 헤더이 csrf 값을 설정 */
          xhr.setRequestHeader(header, token); // <<== 토큰 값 전송
        },
        dataType : "json",
        cache: false,
        success: function(result, status){
          
        },
        error : function (jqXHR, status, error){
          
        }
      });
    }
```
<br/><br/><br/>

# Configuration(스프링 설정 관련)
## WebMvcConfigurer(i)
- org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
- addResourceHandlers() 를 통해 자신의 로컬 컴퓨터에서 파일을 찾을 위치를 설정.
```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry){
    registry.addResourceHandler("/images/**") // url에 /images로 시작하는 경우 설정한 폴더를 기준으로 파일 읽어오도록 설정.
            .addResourceLocations(uploadPath); //로컬에 저장된 파일을 읽어올 root 경로 설정.
}
```
<br/><br/><br/>


# DTO(Data Transfer Object)
- **데이터를 주고 받을 때는 Entity 클래스 자체를 반환하면 안되고 데이터 전달용 객체(DTO)를 사용해야함 !!!**
- 디비 설계를 외부에 노출할 필요도 없으며, 요청과 응답 객체가 항상 엔티티와 같지 않기 떄문.
- 상품 등록 :  화면으로 전달 받은 DTO => 엔티티 객체로 변환 
- 상품 조회 :  엔티티 객체 => DTO 객체 변환
## modelmapper(라이브러리)
- 서로 다른 클래스의 값을 필드의 이름과 자료형이 같으면 getter, setter를 통해 값을 복사해서 객체 반환.
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

#### JPQL로 쿼리 작성 시 생성자 이용해서 DTO로 바로 반환
```java
@Query("select new com.shop.dto.CartDetailDto(ci.id, i.itemNm, i.price, ci.count, im.imgUrl) from ...")
List<CartDetailDto> findCartDetailDtoList(Long cartId);
```
- CartDetailDto 리스트를 쿼리 하나로 조회하는 jpql문. 
- 연관관계 매핑을 지연 로딩으로 설정할 경우 엔티티에 매핑된 다른 엔티티를 조회할 때 추가적으로 쿼리문이 실행됨. 따라서 성능 최적화가 필요한 경우 위와 같이 DTO의 생성자를 이용하여 반환값으로 DTO 객체 생성 가능.
- CartDetailDto의 생성자를 이용하여 DTO를 반환할 때는 "new com.shop.dto.CartDetailDto(ci.id, i.itemNm, i.price, ci.count, im.imUrl)"처럼 new 키워드와 해당 DTO의 패키지, 클래스명을 적어줌. 또한 생성자의 파라미터 순서는 DTO 클래스에 명시한 순으로 넣어야함.


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

- Querydsl을 Spring Data Jpa과 함께 사용하기 위해서는 사용자 정의 리포지토리를 정의해야함.
```과정
1. 사용자 정의 인터페이스 작성
2. 사용자 정의 인터페이스 구현
3. JpaRepository<>를 상속한 커스텀Repository에서 Querydsl 구현한 사용자 정의 인터페이스 함께 상속.
```
- 사용자 정의 인터페이스 impl하는 클래스명 끝에 "Impl"붙여줘야 정상 작동
- QueryDsl에서는 BooleanExpression이라는 where 절에서 사용할 수 있는 값을 지원.
- BooleanExpression을 반환하는 메소드를 만들고 해당 조건들을 다른 쿼리를 생성할 때 사용할 수 있기 때문에 중복 코드를 줄일 수 있음.  => 결과값이 null이면 where절에서 해당 조건은 무시됨.

```java
public class ItemRepositoryCustomImpl implements ItemRepositoryCustom{
    private JPAQueryFactory queryFactory; // 동적 쿼리 생성을 위한 팩토리 클래스

    public ItemRepositoryCustomImpl(EntityManager em){
        this.queryFactory = new JPAQueryFactory(em); // 생성자로 EntityManager를 넣어줌
    }
    ...
    @Override
    public Page<Item> getAdminItemPage(ItemSearchDto itemSearchDto, Pageable pageable) {
        QueryResults<Item> results = queryFactory.selectFrom(QItem.item) //엔티티 지정
                .where(regDtsAfter(itemSearchDto.getSearchDateType()), //where절엔 BooleanExpression 반환하는 조건문을 넣어줌. ","단위로 넣어줄 경우 and 조건으로 인식
                        searchSellStatusEq(itemSearchDto.getSearchSellStatus()),
                        searchByLike(itemSearchDto.getSearchBy(),
                                itemSearchDto.getSearchQuery()))
                .orderBy(QItem.item.id.desc())
                .offset(pageable.getOffset()) //데이터를 가지고올 시작인덱스
                .limit(pageable.getPageSize()) //한번에 가지고 올 최대 갯수
                .fetchResults(); // 조회한 리스트 및 전체 개수 포함하는 QueryResults 반환 // 리스트 조회 및 전체 개수 조회하는 2번의 쿼리문이 실행됨
        List<Item> content = results.getResults();
        long total = results.getTotal();
        return new PageImpl<>(content, pageable, total); //Page 클래스 구현체인 PageImpl 객체 반환.
    }
}
```
```
//Querydsl 조회 결과 반환 메소드
QueryResults<T> fetchResults() //조회 대상 리스트 및 전체 개수 포함
List<T> fetch() // 리스트 
T fetchOne() // 조회 대산 1건이면 반환, 1건 이상이면 에러
T fetchFirst() // 1건만 반환
long fetchCount() // 전체 개수 반환 (count 쿼리)
```

#### @QueryProjection
- Querydsl 결과 조회 시 Dto 객체로 바로 받아볼 수 있음.
- QDto 파일 생성되어 있어야함. 

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


## JPA N+1 문제
- 연관 관계에서 발생하는 이슈로 연관 관계가 설정된 엔티티를 조회할 경우에 조회한 데이터 갯수(n)만큼 연관관계를 조회 쿼리가 추가로 발생하여 데이터를 읽어오게 된다. 이를 n+1문제라고 함.


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

## Principal(i)
- 엔티티를 유니크하게 식별할 때 구현하는 인터페이스
- 스프링 시큐리티에서 사용될 때 users' identity를 식별하는 용도로 사용된다. 

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
