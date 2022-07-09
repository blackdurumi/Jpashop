# Jpashop

## 참고 자료

[실전! 스프링 부트와 JPA 활용 1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/)<br>
[실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94#)<br>

## 배운 내용

### 활용 1편

1. Entity 설계 및 개발
    1. 연관관계 mapping
        - foreign key가 있는 곳이 연관관계의 주인
        - cascade를 설정하면 연관관계가 있는 엔티티 간에 영속성이 전이됨
    2. 연관관계 메소드
        - 연관관계가 있는 Entity의 변경사항을 같이 처리해주는 메소드를 만들자
    3. 모든 연관관계는 지연로딩(FetchType.EAGER)으로 설정해야 함
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
    3. **Form 객체** 활용하여 화면 계층과 서비스 계층을 분리; Entity를 직접 사용하면 안됨
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

1. API 요청 값으로 Entity 자체를 받아오는 것이 아닌, API 스펙에 맞게 DTO를 만들어 받아야 함