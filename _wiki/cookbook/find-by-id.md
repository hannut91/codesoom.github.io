---
layout  : wiki
title   : findById 실행 시 1개 이상의 row가 반환되는 문제
date    : 2022-10-14 17:11:00 +0900
updated : 2022-10-14 17:11:00 +0900
author  : 한지영, 한윤석
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

# 문제

한 User가 2번의 예약을 했을 때, 예약 목록 화면에서 예약 내역 2개가 보이는 것을 기대했는데,
아래와 같은 오류가 발생하며 예약 목록 조회에 실패하고 있었습니다.

```bash
2022-10-14 08:17:43.006 ERROR 1 --- [nio-8080-exec-8] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.orm.jpa.JpaSystemException: More than one row with the given identifier was found: 49, for class: com.codesoom.myseat.domain.User; nested exception is org.hibernate.HibernateException: More than one row with the given identifier was found: 49, for class: com.codesoom.myseat.domain.User] with root cause

org.hibernate.HibernateException: More than one row with the given identifier was found: 49, for class: com.codesoom.myseat.domain.User
...
```

예약 목록 조회 요청 시 호출되는 컨트롤러 메서드는 크게 세 가지 과정으로 이루어져 있습니다.

1. 요청을 한 User의 정보를 얻어옴
2. 예약 목록들을 불러옴
3. 불러온 예약 목록들을 응답 객체로 만들어서 반환

```java
// 예약 목록 조회 요청 컨트롤러 메서드
@PreAuthorize("isAuthenticated()")
@ResponseStatus(HttpStatus.OK)
@GetMapping
public ReservationListResponse reservations(
        @AuthenticationPrincipal UserAuthentication principal
) {
    User user = userService.findById(principal.getId()); // 1

    return new ReservationListResponse(
            service.reservations(user.getId()) // 2
            .stream()
            .map(ReservationResponse::new)
            .collect(Collectors.toList())
    ); // 3
}
```

오류가 발생했음을 인지한 후 디버깅을 해보았고, 1번 과정(요청을 한 User의 정보 얻어오기)에서 문제가 생겼다는 것을 알게 되었습니다.

컨트롤러에서 호출하는 `userService.findById()`는 아래와 같이 단순히 UserRepository의 findById() 메서드의 반환값 User을 그대로 반환하며,

```java
public User findById(
        Long id
) {
    return userRepo.findById(id)
            .orElseThrow(() -> new UserNotFoundException());
}
```

여기서 반환되는 User는 아래와 같이 Repository와 일대일 양방향 관계를 맺고 있었습니다.

```jsx
// 사용자 Entity
public class User {
    @Id
    private Long id;

		@OneToOne(mappedBy = "user")
    private Reservation reservation;
}

// 예약 Entity
public class Reservation {
    @Id
    private Long id;

    @OneToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

어쨌든 문제의 근원지는 `UserRepository.findById()` 라는 것을 알게 됐습니다.
그런데 중복될 수 없는 기본키를 `findById()` 의 매개변수로 주기 때문에 당연히 반환되는 row가 1개일 것으로 예상했고, 에러 메시지의 내용에 의문이 들었습니다. 그래서 findById()가 호출됐을 때 로그에 찍힌 쿼리를 직접 실행시켜봤습니다. 2개의 row가 반환되고 있었습니다.

# 원인

---

이러한 현상을 이해하기 위해서는 먼저 Hibernate의 fetch 속성을 알아야 합니다.
fetch 속성에는 EAGER과 LAZY가 있는데, 두 속성의 주된 차이점은 **데이터가 메모리에 로드되는 순간**입니다.

**EAGER**는 User 데이터를 불러올 때 User와 관련된 Reservation 데이터들도 한꺼번에 로드되어 메모리에 저장됩니다.

**LAZY**는 User 데이터를 모두 불러올 때까지 Reservation 데이터가 초기화 되지 않고 메모리에 로드되지 않습니다.

기존에는 User와 Reservation 엔티티를 일대일 양방향 관계를 맺도록 했었습니다. 이러한 관계에 놓였을 때 어떤 문제가 발생하는지 살펴보도록 하겠습니다.

엔티티간 매핑을 해줄 때 **fetch** 속성을 지정해주지 않으면, 자동으로 EAGER 속성이 적용됩니다.

이때 연관된 데이터들을 한번에 불러오기 위해 left outer join이 이뤄집니다.

```bash
select
        user0_.user_id as user_id1_4_0_,
        user0_.email as email2_4_0_,
        user0_.name as name3_4_0_,
        user0_.password as password4_4_0_,
        reservatio1_.reservation_id as reservat1_1_1_,
        reservatio1_.date as date2_1_1_,
        reservatio1_.status as status3_1_1_,
        reservatio1_.user_id as user_id4_1_1_ 
    from
        user user0_ 
    left outer join
        reservation reservatio1_ 
            on user0_.user_id=reservatio1_.user_id 
    where
        user0_.user_id=?
```

---

### 참고) left outer join이란?

left outer join은 left table을 right table에 포함시키며, left table 중 join 조건을 만족하지 않는 컬럼들도 right table에 포함시킵니다.

Student(left table)

| name | email |
| --- | --- |
| 김철수 | soo@email.com |
| 김영희 | young@email.com |
| 홍길동 | hong@email.com |

Schedule(right table)

| name | title |
| --- | --- |
| 김철수 | 밥먹기 |
| 김철수 | 코테풀기 |
| 김철수 | 자바 공부 |
| 김영희 | 깃 공부 |

3 명의 학생이 Student 테이블에 저장되어 있고,
김철수 학생이 3개의 스케줄을, 김영희 학생이 1개의 스케줄을 등록한 상태를 가정해봅시다.

```sql
select
	s.email, -- 학생 테이블의 이메일 컬럼
	s.name, -- 학생 테이블의 이름 컬럼
	sd.name, -- 스케줄 테이블의 이름 컬럼
	sd.title -- 스케줄 테이블의 제목 컬럼
from
    student s
        left outer join
    schedule sd
    on s.name=sd.name; -- name 컬럼이 일치하는 것들을 join
```

두 테이블에 대해 left outer join을 실행한 결과는 아래와 같습니다.

| s.email | s.name | sd.name | sd.title |
| --- | --- | --- | --- |
| soo@email.com | 김철수 | 김철수 | 밥먹기 |
| soo@email.com | 김철수 | 김철수 | 코테풀기 |
| soo@email.com | 김철수 | 김철수 | 자바 공부 |
| young@email.com | 김영희 | 김영희 | 깃 공부 |
| hong@email.com | 홍길동 | null | null |

Student(left table)가 Schedule(right table)에 포함되었습니다.
이때 홍길동 학생에 대한 Schedule 정보는 없으므로 null로 표시됩니다.

---

다시 돌아와서, 엔티티를 매핑할 때 fetch 속성을 별도로 지정해주지 않아서 자동으로 EAGER 속성이 적용되었고, `findById()` 메서드 실행 시 Reservation과 left outer join이 이뤄져서 총 2개의 row가 반환된 것이죠.

fetch 속성을 LAZY로 설정해보면 어떨까요?
두 엔티티 매핑 시 fetch 속성을 LAZY로 설정한 후 `findById()` 를 실행해보겠습니다

```sql
select
        user0_.user_id as user_id1_6_0_,
        user0_.email as email2_6_0_,
        user0_.name as name3_6_0_,
        user0_.password as password4_6_0_ 
    from
        user user0_ 
    where
        user0_.user_id=?
```

연관된 테이블은 제외하고 오로지 User 정보만 먼저 조회합니다.

```sql
select
        reservatio0_.reservation_id as reservat1_1_0_,
        reservatio0_.date as date2_1_0_,
        reservatio0_.status as status3_1_0_,
        reservatio0_.user_id as user_id4_1_0_ 
    from
        reservation reservatio0_ 
    where
        reservatio0_.user_id=?
```

이후에 User에 연관된 Reservation을 따로 조회합니다.

join은 이뤄지지 않았으나 여전히 불필요한 Reservation까지 조회하고 있습니다.
왜냐하면 처음에 말했듯이 EAGER과 LAZY의 차이는 ‘데이터가 메모리에 언제 적재되느냐’의 차이만 있지, 결과적으로 적재되는 데이터는  같기 때문입니다.

# 해결방법

User가 Reservation을 알고있는 이상, `findById()`는 필연적으로 Reservation을 함께 불러오게 됩니다. 그러나 Reservation 정보는 필요 없기 때문에 오로지 User 데이터만 불러올 수 있도록 해야 합니다.
따라서 User가 Reservation을 알지 못하도록 연관 관계를 단방향으로 설정함으로써 이를 해결할 수 있습니다.

```java
// 사용자 Entity
public class User {
    @Id
    private Long id;

		// - @OneToOne(mappedBy = "user")
    // - private Reservation reservation;
}

// 예약 Entity
public class Reservation {
    @Id
    private Long id;

    @OneToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

기존에 User 엔티티에서 Reservation을 매핑했던 코드를 제거하면,
Reservation에서는 User에 접근 가능하지만 User에서는 Reservation에 접근하지 못하게 됩니다.

# 참고 자료

- [Eager/Lazy Loading In Hibernate](https://www.baeldung.com/hibernate-lazy-eager-loading)
- [HIBERNATE Community Documentation](https://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/ch20.html#performance-fetching)
- [Hibernate: More than one row with the given identifier was found error](https://stackoverflow.com/questions/24210478/hibernate-more-than-one-row-with-the-given-identifier-was-found-error)
- 이상구, 장재영, 김한준, 정재헌 저 / 데이터베이스의 이해
