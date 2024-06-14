# Transactional에 대해

## 서론
> Spring에서 사용되는 Transactional에 대한 면접 질문이 들어와 막상 대답하려하니 얼버무리게 대답하여 제대로 공부하고자 내부적으로 어떤 구조가 존재하고 트랜잭션 성질 등 한번 파헤쳐보려고 한다. 파헤쳐보고 이후 격리 수준에 따른 레벨 지정 및 비관적/낙관적 락에 대한 이야기를 해보려한다.
그리고 Spring 프로젝트를 개발할 시 막연히 @Transactional을 붙여서 Begin(트랜잭션의 시작), Commit(트랜잭션 반영)과 Rollback(에러 발생시 원복)인 구조로, 또는 트랜잭션에서 읽기 전용을 하기 위해 조회용 클래스 또는 메서드에 `readonly = true`를 붙여 사용하곤 했었다. 누군가에게도 암묵적 합의하에 그렇게 사용하고 있다 생각한다.
우선적으로 Transactional에 대해 알아보고 꼬리에 꼬리를 무는 형식으로 포스팅을 진행하려 한다.

간단하게라도 트랜잭션의 4가지 성질을 정의하고자 한다.

> **1. 원자성(Atomicity)**: 한 트랜잭션 내에서 실행한 작업들은 하나의 단위로 처리한다. 즉, 모두 성공 또는 모두 실패.
**2. 일관성(Consistency)**: 트랜잭션은 일관성 있는 데이타베이스 상태를 유지한다. (data integrity 만족 등.)
**3. 격리성(Isolation)**: 동시에 실행되는 트랜잭션들이 서로 영향을 미치지 않도록 격리해야한다.
**4. 영속성(Durability)**: 트랜잭션을 성공적으로 마치면 결과가 항상 저장되어야 한다.
필자는 격리성과 일관성이 가장 중요하다 생각하는데, 4가지 다 중요한 성질이다.

---
# @Transactional
해당 어노테이션은 무엇인가? **데이터베이스 작업에서 트랜잭션을 관리하는 데에 사용되는 어노테이션**이라고 한 줄 요약이 가능하다.
트랜잭션을 관리하는 선언적 방법을 제공하며 트랜잭션 어노테이션은 클래스 또는 메서드 레벨에 적용을 할 수 있다. 
### @Transactional 우선순위는 어떻게 되는가?

1. 클래스 메소드
2. 클래스
3. 인터페이스 메소드
4. 인터페이스

클래스 수준에서 적용하면 클래스의 모든 메서드에 대한 기본 트랜잭션 설정을 설정하고,
메서드 수준에서 적용하면 해당 특정 메서드의 class-level settings 값을 overrides한다.

단 주의해야 할 부분이 있다. @Transactional 어노테이션 같은 경우 Spring AOP를 이용하게 되는데 이 AOP는 기본적으로 Dynamic Proxy를 이용한다. Dynamic Proxy는 인터페이스 기반으로 동작하기 때문에 인터페이스가 없을경우 트랜잭션이 동작하지 않는다.

인터페이스 없이 트랜잭션 동작하게 하려면 CGLib(Code Generation Library) Proxy를 이용하면 된다. CGLib Proxy는 클래스에 대한 Proxy가 가능하기 때문에 인터페이스가 없어도 된다.

이처럼 Transactional은 Proxy 방식으로 동작되어 Proxy 객체를 생성한다.
Proxy 객체는 해당 메서드 실행 이전에 PlatformTransactionManager를 사용하여 트랜잭션의 시작부터 결과까지 Commit 또는 Rollback을 진행한다.

#### Commit과 Rollback은 어느 경우에 발생하는가?
CheckedException 또는 예외가 없을 때는 Commit
UncheckedException이 발생하면 Rollback

## Transactional의 옵션
```java
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    String[] label() default {};

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    String timeoutString() default "";

    boolean readOnly() default false;

    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}
```

해당 코드는 직접 Transactional을 직접 까봤다.

> **1. isolation**: 트랜잭션에서 일관성없는 데이터 허용 수준을 설정한다.
**2. propagation**: 트랜잭션 동작 도중 다른 트랜잭션을 호출할 때, 어떻게 할 것인지 지정하는 옵션이다.
**3. noRollbackFor**: 특정 예외 발생 시 rollback하지 않는다.
**4. rollbackFor**: 반대로 특정 예외 발생 시 rollback한다.
**5. noRollbackForClassName**: 특정 클래스 배열 예외 발생 시 rollback하지 않는다.
**6. rollbackForClassName**: 반대로 특정 클래스 배열 예외 발생 시 rollback한다.
**7. timeout**: 지정한 시간 내에 메소드 수행이 완료되지 않으면 rollback 한다. (-1일 경우 timeout을 사용하지 않는다. default가 -1이기에)
**8. readOnly**: 트랜잭션을 읽기 전용에 대한 설정 여부를 지정한다. (default는 false이다.)
**9. value**: 사용할 트랜잭션 관리자를 지정한다.

이와 같이 다양한 옵션을 제공하고 있다.
이 중에서 readOnly, isolation 옵션에 대해 알아보자. 그 외는 쉽게 유추하거나 구글링하면 나오는 정보가 많다.

---
## 1. isolation 옵션
> DEFAULT : 기본 격리 수준이며, DB의 lsolation Level을 따른다.

> READ_UNCOMMITED (level 0) : 커밋되지 않는 데이터에 대한 읽기를 허용
어떤 사용자가 A라는 데이터를 B라는 데이터로 변경하는 동안 다른 사용자는 B라는 아직 완료되지 않은(Uncommitted 혹은 Dirty)데이터 B를 읽을 수 있다.

> READ_COMMITED (level 1) : 커밋된 데이터에 대해 읽기 허용
어떠한 사용자가 A라는 데이터를 B라는 데이터로 변경하는 동안 다른 사용자는 해당 데이터에 접근할 수 없다.

> REPEATEABLE_READ (level 2) : 동일 필드에 대해 다중 접근 시 모두 동일한 결과를 보장
트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 shared lock이 걸리므로 다른 사용자는 그 영역에 해당되는 데이터에 대한 수정이 불가능하다.
선행 트랜잭션이 읽은 데이터는 트랜잭션이 종료될 때까지 후행 트랜잭션이 갱신하거나 삭제가 불가능 하기 때문에 같은 데이터를 두 번 쿼리했을 때 일관성 있는 결과를 리턴한다.

> SERIALIZABLE (level 3) : 가장 높은 격리, 성능 저하의 우려가 있음
트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 shared lock이 걸리므로 다른 사용자는 그 영역에 해당되는 데이터에 대한 수정 및 입력이 불가능하다.

이처럼 isolation에서 트랜잭션에 대한 격리수준을 지정하고 데이터 정합성에 대한 일관성, 독립성을 분간할 수 있다.


## 격리수준에 따른 문제

| IsolationLevel | Dirty Read | Non-Repeatable Read | Phantom Read |
| :---- | ------ | ------ | ------ |
| Read Uncommitted | O | O | O |
| Read Committed | - | O | O |
| Repeatable Read | - | - | O |
| Serializable | - | - | - |



> **Dirty Read**
트랜잭션 1이 수정중인 데이터를 트랜잭션 2가 읽을 수 있다. 만약 드랜잭션 1의 작업이 정상 커밋되지 않아 롤백되면, 트랜잭션 2가 읽었던 데이터는 잘못된 데이터가 되는 것이다.
(데이터 정합성에 어긋남)

> **Non-repeatable read**
트랜잭션 1이 회원 A를 조회중에 트랜잭션 2가 회원 A의 정보를 수정하고 커밋한다면, 트랜잭션 1이 다시 회원A를 조회했을 때는 수정된 데이터가 조회된다. (이전 정보를 다시 조회할 수 없음). 이처럼 반복해서 같은 데이터를 읽을 수 없는 경우이다.

> **Phantom read**
트랜잭션 1이 10살 이하의 회원을 조회했는데 트랜잭션 2가 5살 회원을 추가하고 커밋하면 트랜잭션 1이 다시 10살 이하 회원을 조회했을 때 회원 한명이 추가된 상태로 조회된다. 이처럼 반복 조회시 결과집합이 달라지는 경우이다.

> **트랜잭션 격리 수준의 필요성**
당연히 레벨이 높아질 수록 데이터 무결성을 유지할 수 있다.
하지만, 무조건적인 상위 레벨을 사용할 시 Locking으로 동시에 수행되는 많은 트랜잭션들이 순차적으로 처리하게 되면서 DB의 성능은 떨어지게 되고 비용이 높아진다.
그렇다고 Locking의 범위를 줄이게 되면 잘못된 값이 처리될 여지도 발생한다.
그러므로 최대한 효율적인 방안을 찾아 상황에 맞게 사용하는 것이 중요하다.

트랜잭션의 격리 수준에 대해서는 따로 알아보도록 할 것이다.

---
## 2. propagation (전파속성)
적용 방법
```java
@Transactional(propagation = Propagation.REQUIRED)
public void addUser(UserDTO dto) throws Exception {
	// 로직 구현
}
```

> REQUIRED (Defualt)
이미 진행중인 트랜잭션이 있다면 해당 트랜잭션 속성을 따르고, 진행중이 아니라면 새로운 트랜잭션을 생성한다.

> REQUIRES_NEW
항생 새로운 트랜잭션을 생성한다. 이미 진행중인 트랜잭션이 있다면 잠깐 보류하고 해당 트랜잭션 작업을 먼저 진행한다

> SUPPORT
이미 진행 중인 트랜잭션이 있다면 해당 트랜잭션 속성을 따르고, 없다면 트랜잭션을 설정하지 않는다.

> NOT_SUPPORT
이미 진행중인 트랜잭션이 있다면 보류하고, 트랜잭션 없이 작업을 수행한다.

> MANDATORY
이미 진행중인 트랜잭션이 있어야만, 작업을 수행한다. 없다면 Exception을 발생시킨다.

> NEVER
트랜잭션이 진행중이지 않을 때 작업을 수행한다. 트랜잭션이 있다면 Exception을 발생시킨다.

> NESTED
진행중인 트랜잭션이 있다면 중첩된 트랜잭션이 실행되며, 존재하지 않으면 REQUIRED와 동일하게 실행된다.

---
## 3. noRollbackFor (예외무시)
특정 예외 발생 시 Rollback 처리 하지 않음.

적용 방법
```java
@Transactional(noRollbackFor = Exception.class)
public void addUser(UserDTO dto) throws Exception {
	// 로직 구현
}
```

---
## 4. rollbackFor (예외추가)
특정 예외 발생 시 강제로 Rollback

적용 방법
```java
@Transactional(rollbackFor = Exception.class)
public void addUser(UserDTO dto) throws Exception {
	// 로직 구현
}
```

> 그래서 왜 (rollbackFor = Exception.class)를 하는 것인가?
@Transactional은 기본적으로 Unchecked Exception, Error 만을 rollback하고 있다.
그렇기 때문에 모든 예외에 대해서 rollback을 진행하고 싶을 경우 (rollbackFor = Exception.class) 를 붙여야 한다는 것을 깨달았다.
그 이유는 Checked Exception 같은 경우는 예상된 에러이며 Unchecked Exception, Error 같은 경우는 예상치 못한 에러이기 때문이란 것을 알게 되었다.

---
## 5. timeout (시간지정)
지정한 시간 내에 해당 메소드 수행이 완료되지 않을 경우 rollback 수행
-1일 경우 no timeout, Default = -1

적용방법
```java
@Transactional(timeout = 10)
public void addUser(UserDTO dto) throws Exception {
	// 로직 구현
}
```

---
## 6. readOnly (읽기 전용)
true 시 insert, update, delete 실행 시 예외 발생
Default = false

적용방법
```java
@Transactional(readonly = true)
public void addUser(UserDTO dto) throws Exception {
	// 로직 구현
}
```

사실 readonly에 대해 더 알아보고 싶었다만, 글이 길어져 포스팅 하는대로 링크를 달 것입니다.


## 마무리
이번에 Transactional의 간단한 정의, 몇몇 옵션에 대해서 간단히 알아보았다.
앞 서론에서 작성했듯이 더 작성할 내용이 너무 많아서 쪼개서 꼬리에 꼬리를 무는 방식으로 포스팅하도록 하겠습니당



## Reference
- https://tecoble.techcourse.co.kr/post/2021-05-25-transactional/
- https://incheol-jung.gitbook.io/docs/q-and-a/spring/transactional

