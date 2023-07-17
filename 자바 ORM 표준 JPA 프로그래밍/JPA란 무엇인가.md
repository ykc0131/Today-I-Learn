# JPA란 무엇인가

### | 왜 테이블 설계는 열심히 하면서 제대로 된 객체 모델링은 하지 않을까?

### JPA
- 자바 진영의 ORM 기술 표준. <br> *객체와 관계형 데이터베이스간의 차이를 중간에서 해결해주는 ORM 프레임워크. <br>
- 지루하고 반복적인 CRUD SQL을 알아서 처리해줌.
- 객체 모델링과 관계형 데이터베이스 사이의 차이점도 해결해준다. 

#### 💡 JPA를 통해서 개발자는 직접 SQL을 작성하는 것이 아닌 어떤 SQL이 실행될지 생각만 하면 된다! 
<br>

### 개발자는 SQL 매퍼가 아니다.
> 프로젝트 참여할 때마다 SQL작성이 시간을 꽤 들여서 힘들었던 경험이 있다. JPA를 사용해서 더 좋은 객체 모델링과 테스트 작성에 공을 들여야겠다.!
<br>

## 1.1 SQL을 직접 다룰 때 발생하는 문제점 
#### Entity (엔티티) = 비즈니스 요구사항을 모델링한 객체

SQL에 모든 것을 의존하는 상황에서는 개발자들이 엔티티를 신뢰하고 사용할 수 없다 .

### 애플리케이션에서 SQL을 직접 다룰 때 발생하는 문제점 
- 진정한 의미의 계층 분할이 어렵다. 
  - 물리적으로 SQL과 JDBC API를 데이터 접근 계층에 숨기는데 성공했을지라도 논리적으로 엔티티와 아주 강한 의존관계를 가지고 있다. 
    <br> - 어떤 SQL이 실행되고 어떤 객체들이 조회되는지 일일이 확인해야함.
- 엔티티를 신뢰할 수 없다 .
- SQL에 의존적인 개발을 피하기 어렵다. 

### JPA 간단한 CRUD API
- 저장 기능
  ~~~java
  jpa.persist(member);
  ~~~
  
- 조회 기능
  ~~~java
  jpa.find(Member.class,id);
  ~~~
    JPA는 객체와 매핑정보를 보고 적절한 SELECT SQL을 생성해서 데이터베이스에 전달하고 그 결과로 객체를 생성해서 반환한다.
  
- 수정 기능 
  ~~~java
  Member member = jpa.find(Member.class,id);
  member.setName("이름변경");
  ~~~
    JPA는 별도의 수정 메소드를 제공하지 않는다. <br>
    대신, 객체를 조회해서 값을 변경만 하면 트랜잭션 커밋 시 적절한 UPDATE SQL이 전달된다.
  
- 연관된 객체 조회
  ~~~java
  Member member = jpa.find(Member.class,id);
  Team team = member.getTeam();
  ~~~
    JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다. 
<br>

## 1.2 패러다임의 불일치
  
애플리케이션은 <strong>객체지향 언어</strong><br>
데이터는 <strong>관계형 데이터베이스</strong><br>
#### &rarr; 패러다임 불일치

1. 상속
    ~~~java
    abstract class Item{
      Long id;
      String name;
      int price;
    }
    
    //상속
    class Album extends Item{};
    
    class Movie extends Item{};
    
    class Book extends Item{};
    ~~~
    
    JDBC API를 사용하면 부모 객체에서 부모 데이터를 꺼내 ITEM용 INSERT SQL을 작성하고 자식 객체에서 자식 데이터만 꺼내서 INSERT SQL을 작성해야한다.
    
    #### &rarr; JPA는 상속과 관련된 패러다임 불일치 문제를 개발자 대신 해결해준다. 
    
    - 저장
    ~~~java
    jpa.persist(album);
    ~~~
    persist() 메소드를 사용해서 객체를 저장하면 JPA가 다음 SQL을 실행해서 두 테이블에 나누어 저장한다.
    ~~~sql
    INSERT INTO ITEM ...
    INSERT INTO ALBUM ...
    ~~~
    
    - 조회
    ~~~java
    String albumId = "id100";
    Album album = jpa.find(Album.class,albumId);
    ~~~
    JPA는 ITEAM과 ALBUM 두 테이블을 조인해서 필요한 데이터를 조회하고 그 결과를 반환한다. 
    ~~~sql
    SELECT I.*,A.*
      FROM ITEM 
      JOIN ALNUM A ON I.ITEM_ID = A.ITEM_ID
    ~~~
    
2. 연관관계
    ~~~sql
    CREATE TABLE MEMBER (
    MEMBER_ID,
    TEAM_ID,
    USERNAME,
    PRIMARY KEY (MEMBER_ID)
    );
  
    CREATE TABLE TEAM (
    TEAM_ID,
    NAME,
    PRIMARY KEY(TEAM_ID)
    );
    ~~~
    #### 객체지향 모델링
    객체는 <strong>참조</strong>를 사용해서 연관된 객체를 조회하고 테이블은 <strong>외래 키</strong>로 JOIN을 이용해서 연관된 테이블을 조회한다.
    객체를 데이터베이스에 저장하려면 team 필드를 TEAM_ID 외래 키 값으로 변환해야하는데 이를 JPA가 처리해준다. <br>
    #### &rarr; JPA는 연관관계과 관련된 패러다임 불일치 문제를 해결해준다. 
    ~~~java
    class Member{
      String id;  // MEMBER_ID 컬럼 사용
      Team team;  // 참조로 연관관계를 맺는다.
      String username; // USERNAME 컬럼 사용
      
      Team getTeam(){
        return team;
      }
    }
    
    class Team{
      Long id;   // TEAM_ID PK 사용
      String name; // NAME 컬럼 
    }
    ~~~
    
    - 저장
    ~~~java
    member.setTeam(team); // 회원과 팀 연관관계 설정
    jpa.persist(member); //회원과 연관관계 함께 저장
    ~~~
    회원과 팀의 관계를 설정하고 객체를 저장하면 된다. JPA는 team의 참조를 외래 키로 변환해서 INSERT SQL을 데이터베이스에 전달한다. 
    
    - 조회
    ~~~java
    Member member = jpa.find(Member.class, memberId);
    Team team = member.getTeam();
    ~~~
3. 객체 그래프 탐색
    <br><br>어디까지 객체 그래프 탐색이 가능한지 알아보려면 데이터 접근 계층인 DAO를 열어서 SQL을 직접 확인해야한다. 
    그렇다고 member와 연관된 모든 객체 그래프를 데이터베이스에서 조회해서 애플리케이션 메모리에 올려두는 것은 현실성이 없다. 
    ~~~java
    memberDAO.getMember();
    memberDAO.getMemberWithTeam();
    memberDAO.getMemberWithOrderWithDelivery(); ...
    ~~~
    
    #### &rarr; JPA를 사용하면 객체 그래프를 마음껏 탐색할 수 있다. 
    JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다. &rarr; 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다.
    따라서, 이를 <strong>지연로딩</strong> 이라고 한다. 
    
    ~~~java
    class Member{
      private Order order;
      
      public Order getOrder(){
        return order;
      }
    }
    ~~~
    
    실제 Order 객체를 사용하는 시점에 JPA는 데이터베이스에서 ORDER 테이블을 조회한다.
    ~~~java
    // 처음 조회 시점에 SELECT MEMBER SQL
    Member member = jpa.find(Member.clas, memberId);
    
    Order order = member.getOrder();
    order.getOrderDate(); // Order를 사용하는 시점에 SELECT ORDER SQL
    ~~~
    
    혹은, JPA는 연관된 객체를 즉시 함께 조회할지 아니면 실제 사용되는 시점에 지연해서 조회할지를 간단한 설정으로 정의할 수 있다.
    
4. 비교
    ~~~java
    String memberId = "100";
    Member member1 = memberDAO.getMember(memberId);
    Member member2 = memberDAO.getMember(memberId);
    
    member1 == member2; // false
    ~~~
    객체 측면에서는 MemberDAO.getMember()를 호출할 때마다 new Member()로 인스턴스가 새로 생성되므로 둘은 다른 인스턴스이기 때문에 false가 반환된다. 
    
    #### &rarr; JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다. 
    
    ~~~java
    String memberId = "100";
    Member member1 = jpa.find(Member.id, memberId);
    Member member2 = jpa.find(Member.id, memberId);
    
    member1 == member2; // true
    ~~~


## 1.3 JPA란 무엇인가?

### JPA 
JPA(Java Persistence API) = 자바 진영의 <strong>ORM</strong> 기술 표준
JPA는 애플리케이션과 JDBC 사이에서 동작한다. 

### ORM 
ORM(Object-Relational Mapping) 프레임워크 = 객체와 관계형 데이터베이스를 매핑
&rarr; ORM 프레임워크는 객체와 테이블을 매핑해서 패러다임의 불일치 문제를 개발자 대신 해결해준다. 

- 저장
  - DAO에서 Entity 객체 persist() <br>
  - ORM 프레임워크 
    - Entity 분석
    - INSERT SQL 생성
    - JDBC API 사용 &rarr; DB에 객체 저장

- 조회<br>
  - DAO에서 find()<br>
  - ORM 프레임워크<br>
      - SELECT SQL 생성
      - JDBC API 사용
      - ResultSet 매핑 &rarr; DB에 SQL 사용 
      - 결과 반환 후 Entity 객체 return
#### &rarr; 패러다임 불일치 해결
  
ORM 프레임워크는 단순히 SQL을 개발자 대신 생성해서 데이터베이스에 전달해주는 것뿐만 아니라 다양한 패러다임의 불일치 문제들도 해결해준다. 


## ⭐ JPA를 사용하는 이유! ⭐
- 생산성<br>
      데이터베이스 설계 중심의 패러다임 &rarr; <strong>객체 설계 중심의 패러다임</strong><br>
- 유지보수<br>
      SQL과 JDBC API 코드를 <strong>JPA가 대신 처리</strong>해주므로 유지보수해야 하는 코드 수가 줄어든다.<br>
- 패러다임의 불일치 해결<br>
- 성능<br>
- 데이터 접근 추상화와 벤더 독립성<br>
      관계형 데이터베이스는 같은 기능도 벤더마다 사용법이 다른 경우가 많다. JPA는 애플리케이션과 데이터베이스 사이에 추상화된 데이터 접근 계층을 제공해서 특정 데이터베이스 기술에 종속되지 않도록 한다.<br>
- 표준<br>
      JPA는 자바 진영의 ORM 기술 표준이다. 표준을 사용하면 다른 구현 기술로 손쉽게 변경할 수 있다. 

  
