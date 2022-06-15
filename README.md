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
- 조건이 많아질 때 쿼리 메소드를 선언하면 이름이 정말 길어지기도 함. 복잦ㅂ한 쿼리를 다루기엔 부적합.
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

