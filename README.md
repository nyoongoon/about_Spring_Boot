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
flush()
```

## 쿼리 메소드
- find + (엔티티 이름(생략가능)) + By + 변수이름
- 조건이 많아질 때 쿼리 메소드를 선언하면 이름이 정말 길어지기도 함. 복잡한 쿼리를 다루기엔 부적합.

## JPAQuery(JPQL) 
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

