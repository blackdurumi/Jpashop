# Jpashop

## 참고 자료

[실전! 스프링 부트와 JPA 활용 1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/)<br>
[실전! 스프링 부트와 JPA 활용 2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94#)<br>

## 배운 내용

### 활용 1편

1. Entity 설계 및 개발
    1. 연관관계 mapping
        - foreign key가 있는 곳이 연관관계의 주인
        - cascade를 설정하면 연관관계가 있는 엔티티 간에 영속성이 전이됨
    2. 연관관계 메소드
        - 연관관계가 있는 Entity의 변경사항을 같이 처리해주는 메소드를 만들자
    3. 모든 연관관계는 지연로딩(FetchType.LAZY)으로 설정해야 함
        - Why? 연관관계가 필요없는 경우에도 항상 데이터를 조회하기 때문에 성능 저하 유발
2. Repository - Service - Controller 계층형 구조에서의 개발
    1. Repository에서는 EntityManager를 통한 데이터 접근, 비즈니스 로직 구현
    2. Service에서는 Repository의 메소드를 이용하여 Entity에 필요한 것을 요청하는 메소드 구현
    3. Controller에서는 기능을 구현하기 위해 Service의 메소드 사용

    - 이러한 구조를 **도메인 모델 패턴**이라고 함
3. Test 코드 작성
    - given, when, then
4. MVC 기초 및 화면 계층
    1. Get/Post mapping
    2. View에 전달할 객체(데이터)를 Model에 보관
    3. **Form 객체** 활용하여 화면 계층과 서비스 계층을 분리; **Entity를 직접 사용하면 안됨**
5. 변경 감지와 병합(merge)
    1. 변경 감지
        1. 수정하려는 Entity를 영속성 컨텍스트로 가져옴
        2. Entity를 수정한 후 트랜잭션 커밋
        3. dirty check하여 update하는 쿼리를 생성하여 날림
    2. 병합
        1. 준영속 Entity에서 id값을 이용해 영속 Entity를 조회
        2. 영속 Entity를 준영속 Entity로 전부 교체

        - 모든 속성이 변경되므로 변경을 원하지 않은 필드도 바뀔 수 있음

    - Entity 변경이 필요할 때는 항상 변경 감지를 사용하자
    - 변경이 필요한 필드만 처리하기 위해 DTO를 이용하자

### 활용 2편

#### 지연 로딩과 조회 성능 최적화

1. API를 개발할 때...

> Entity를 직접 노출시키지 말자

API 요청 값으로 Entity 자체를 받아오는 것이 아닌, API 스펙에 맞게 DTO를 만들어 받아야 함

2. DTO를 만들었더라도...

> N+1 문제가 발생한다<br>
> * 첫 번째 쿼리 조회 결과 N개에 대해 지연로딩이 발생하면 N개의 추가 쿼리 발생
> * 지연로딩은 영속성 컨텍스트를 조회하므로 이미 조회됐던 Entity는 쿼리가 생략됨

fetch join을 사용하자

3. Fetch join

- 연관관계 Entity를 join하여 select할 때 같이 가져옴
- 지연로딩이 있을 일이 없음

4. JPA에서 DTO로 바로 조회

- 쿼리의 select문에 DTO를 넣어 원하는 필드만 가져올 수 있음
- 불필요한 필드를 조회하지 않는다는 장점
    - DB와 app 간 네트워킹 코스트가 줄어들지만, 효과가 크지는 않음
- 재사용성이 떨어진다는 단점
- 전적으로 화면에 의존하는 쿼리와 DTO를 구성하게 되므로 메소드를 일반적인 Repository에 만드는 것이 아닌 쿼리용 레포지토리를 따로 만들자
    - 일반적인 Repository는 순수하게 Entity를 조회하는 용도, fetch join하는 정도의 메소드만 만들자

#### 컬렉션 조회 최적화

1. Entity를 노출시키면 안된다는 기본 원칙
    - 특히 DTO 내부에도 Entity가 있다면 DTO 안에 DTO를 만들어야 할 수도 있음
2. fetch join
    1. N+1 문제 해결 가능하다는 장점
    2. 페이징이 불가능하다는 단점
        - 1대다 join을 하게되면 다 쪽의 row 개수만큼 row가 늘어난다.
            - 따라서 distinct 키워드를 넣어 DB와 app 모두 중복을 제거하게 하자.
        - DB에서 데이터를 읽어오고 메모리에서 페이징을 하는 위험성
    3. 컬렉션 fetch join은 한 번만 사용할 수 있다는 단점
3. 한계 돌파
    * 방법
        1. toOne 관계는 모두 fetch join 한다.
        2. 컬렉션(toMany 관계) 조회는 지연로딩으로 가져온다. 이 때 성능 최적화를 위해
            1. 글로벌 설정 : application.yml -> hibernate.default_batch_fetch_size 설정
            2. 개별 설정 : @BatchSize
    * 효과
        1. 컬렉션을 in을 이용하여 한번에 불러온다. -> 1+N 문제를 1+1로 줄여준다.
        2. DB 데이터 전송량을 줄여준다; 중복된 값 전달이 없어진다.
        3. 페이징이 가능하다.
4. DTO로 직접 조회
    1. 루트 쿼리에서 toOne 관계는 fetch join으로 한번에 불러오고 toMany는 별도 메소드와 DTO로 각각 조회한다.(1+N)
    2. Map을 이용하여 컬렉션 조회에 사용될 필드를 리스트로 전달하여 in 쿼리문으로 한번에 조회할 수 있다.(1+1)
5. 총정리
    1. fetch join과 hibernate.default_batch_fetch_size를 이용한 Entity 조회 방식을 우선 적용
    2. 이후 DTO로 조회하는 방식 도입을 고민하자.

#### OSIV(Open Session In View)

@Transaction이 시작되어 DB connection을 가져오고 transaction이 끝나고 바로 반환하는 것이 아닌,<br>
유저에게 화면이 표시되거나 API 응답이 갈 때까지 connection을 들고 있다가 반환한다.

너무 오랫동안 DB connection resource를 들고있기 때문에 트래픽이 많은 서비스에서는 connection이 모자르게 되고 장애가 발생한다.

OSIV를 끄면 모든 지연로딩을 transaction 내에서 처리해야 한다.
