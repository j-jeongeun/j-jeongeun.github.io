---
title: Transaction 이야기
date: 2023-07-31
categories: [Transaction]
tags: [Transaction]
---

## 1. Transaction이란?

트랜잭션(Transaction)은 작업의 완전성을 보장해주는 것이다. 즉 논리적인 작업 셋을 모두 완벽하게 처리하거나, 처리하지 못할 경우에는 원 상태로 복구해서 작업의 일부만 적용되는 현상(Partial Update)이 발생하지 않게 만들어주는 기능이다.

## 2. Transaction 특징

1. 원자성 (Atomicity)
	- `All or Nothing`
	- 트랜잭션의 연산은 데이터베이스에 모두 반영되거나 전혀 반영되지 않아야 한다.
	- 트랜잭션 내의 모든 명령은 반드시 완벽히 수행되어야 하며, 모두가 완벽히 수행되지 않고 어느 하나라도 오류가 발생하면 트랜잭션 전부가 취소되어야 한다.   
2. 일관성 (Consistency)
	- 트랜잭션이 성공적으로 완료되면 언제나 일관성 있는 데이터베이스를 제공해야 한다.
3. 독립성 (Isolation)
	- 하나의 트랜잭션은 다른 트랜잭션에 끼어들 수 없고 마찬가지로 독립적이어야 한다. 각각의 트랜잭션은 독립적이므로 서로 간섭이 불가능하다.
4. 지속성 (Durability)
	- 트랜잭션이 성공적으로 완료되면 영구적으로 결과에 반영되어야 한다.

## 3. Transaction 격리 수준

격리 수준은 하나의 트랜잭션 내에서 또는 여러 트랜잭션 간의 작업 내용을 어떻게 공유하고 차단할 것인지를 결정하는 레벨을 의미한다. 여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정하는 것이다.

- 격리 수준의 종류

1. READ UNCOMMITTED == DIRTY READ
	- 일반적인 데이터베이스에서는 거의 사용하지 않는다.
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE
	- 동시성이 중요한 데이터베이스에서는 거의 사용되지 않는다.

4개의 격리 수준에서 순서대로 뒤로 갈수록 각 트랜잭션 간의 데이터 격리(고립) 정도가 높아지며, 동시 처리 성능도 떨어지는 것이 일반적이다.

데이터베이스의 격리 수준을 이야기하면 항상 함께 언급되는 세 가지 문제점(Dirty Read, Non-Repeatable Read, Phantom Read)이 있다. 이 문제점들은 격리 수준 레벨에 따라 발생할 수도 있고 발생하지 않을 수도 있다.     

||DIRTY READ|NON-REPEATABLE READ|PHANTOM READ|
|---|---|---|---|
|READ UNCOMMITTED|발생|발생|발생|
|READ COMMITTED|없음|발생|발생|
|REPEATABLE READ|없음|없음|발생(InnoDB는 없음)|
|SERIALIZABLE|없음|없음|없음|

MySQL의 기본 격리 수준은 REPEATABLE READ이고, ORACLE의 경우 READ COMMITTED을 기본값으로 사용하고 있다.

격리 수준에 따라 위의 문제들이 어떻게 발생하지 않고 발생하는지 살펴보자.

### 3.1 READ UNCOMMITTED == DIRTY READ

READ UNCOMMITTED는 그림과 같이 각 트랜잭션에서의 변경 내용이 COMMIT이나 ROLLBACK 여부에 상관없이 다른 트랜잭션에서 보인다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/3bfe7911-67ca-420f-a1f0-52b38aa1b1e0)

COMMIT되기 전 데이터인 Lara의 경우, 사용자B가 조회한다면 Lara의 정보를 조회할 수 있다. 하지만, 만약 사용자A가 어떤 이유로 Lara 데이터를 ROLLBAK할 경우 사용자B는 알지 못한다.

이처럼 어떤 트랜잭션에서 처리한 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수있는 현상을 DIRTY READ라고 한다. DIRTY READ가 발생한다면 사용자들이 해당 데이터를 신뢰할 수 없을 것이다. 그래서 실제로는 최소한 READ COMMITTED 이상의 격리 수준을 사용하라고 권장한다.

### 3.2 READ COMMITTED

READ COMMITTED는 가장 많이 선택되는 격리 수준이다. 이 레벨에서는 DIRTY READ 같은 현상은 발생하지 않는다. 어떤 트랜잭션에서 데이터를 변경했더라도 COMMIT이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있기 때문이다. 

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/ab7015f5-43eb-4fec-bcf0-c2fad2b14fc7)

READ COMMITTED 격리 수준에서도 `NON-REPEATABLE READ`라는 문제가 발생한다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/3f6fbfee-f4f2-49f3-aefd-787725d46a61)

사용자A가 데이터를 변경하기 전에는 일치하는 결과가 없지만 데이터를 변경하고 커밋을 실행하면 1건이 조회 된다.  하나의 트랜잭션 내에서 똑같은 SELECT 쿼리를 실행했을 때는 항상 같은 결과를 가져와야 한다는 `REPEATABLE READ` 정합성에 어긋나는 것이다.

이런 문제는 일반적으로 큰 문제가 없을 수도 있지만, 하나의 트랜잭션에서 동일 데이터를 여러 번 읽고 변경하는 작업이 금전적인 처리와 연결되면 문제가 될 수도 있다.  사용 중인 트랜잭션의 격리 수준에 의해 실행하는 SQL 문장이 어떤 결과를 가져오게 되는지를 정확히 예측할 수 있어야 한다.  

### 3.3 REPEATABLE READ

REPEATABLE READ 격리 수준에서는 `NON-REPEATABLE READ` 문제가 발생하지 않는다. 트랜잭션이 ROLLBACK될 가능성에 대비해 변경되기 전 데이터를 백업 공간에 저장해두고 실제 레코드 값을 변경한다. 이러한, 변경 방식을 MVCC(Multi Version Concurrency Control)이라고 한다.

REPEATABLE READ는 이 MVCC를 위해 저장된 백업 데이터를 이용해 동일 트랜잭션 내에서는 동일한 결과를 보여줄 수 있게 보장한다. 

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/b4a145d3-c997-4f5f-b503-d3488ec905a0)

모든 트랜잭션은 고유한 트랜잭션 번호를 가지며, 백업 데이터에는 변경을 발생시킨 트랜잭션의 번호가 포함되어 있다. 이를 이용하여 한 트랜잭션 내에서 동일한 결과를 보여준다. 

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/839b03d0-e976-4278-9c24-f82c17b37787)

REPEATABLE READ에서는 조회 결과가 동일해야 하지만, SELECT ... FOR UPDATE 쿼리 결과는 다르게 나온다. 이렇게 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다 안 보였다 하는 현상을 `PHATOME READ`라고 한다. SELECT ... FOR UPDATE 쿼리는 SELECT 하는 레코드에 쓰기 잠금을 걸어야 하는데, 백업 데이터에는 잠금을 걸 수 없다. 그래서 SELECT ... FOR UPDATE나 SELECT ... LOCK IN SHARE MODE로 조회되는 레코드는 백업 영역의 변경 전 데이터를 가져오는 것이 아니라 현재 레코드의 값을 가져오게 되는 것이다.

### 3.4 SERIALIZABLE

가장 단순한 격리 수준이면서 동시에 가장 엄격한 격리 수준이다. 그만큼 동시 처리 성능도 다른 트랜잭션 격리 수준보다 떨어진다.

격리 수준이 SERIALIZABLE로 설정되면 읽기 작업도 공유 잠금(읽기 잠금)을 획득해야 하며, 동시에 다른 트랜잭션은 해당 레코드를 변경하지 못하게 된다. 즉, 한 트랜잭션에서 읽고 쓰는 레코드를 다른 트랜잭션에서는 절대 접근할 수 없는 것이다. 하지만, 갭 락과 넥스트 키 락 덕분에 REPEATABLE READ 격리 수준에서도 이미 `PHANTOM READ`가 발생하지 않기 때문에 굳이 SERIALIZABLE을 사용할 필요성은 없어 보인다.

## 4. Transaction 전파 레벨 (with 스프링 @Transactional Annotation)

스프링에서 `@Transactional` Annotation을 제공한다. 해당 Annotation은 진행중인 트랜잭션에서 다른 트랜잭션이 참여할 때의 합류조건을 설정할 수 있는 Propagation 옵션을 제공한다. 데이터베이스에서 자체적으로 제공해주는 트랜잭션 격리 수준과 다르게 전파 레벨은 스프링에서 개발자들의 편의를 위해 제공해주는 기능이다.

- 전파 레벨의 종류
1. REQUIRED
2. REQUIRES_NEW
3. SUPPORTS
4. NOT_SUPPORTED
5. MANDATORY
6. NESTED
7. NEVER

보통은 REQUIRED, REQUIRES_NEW를 가장 많이 사용한다. 스프링은 기본으로 `REQUIRED`을 사용한다.

### 4.1 REQUIRED (default)

기존 트랜잭션이 없으면 새로운 트랜잭션을 만들고, 기존 트랜잭션이 있으면 기존 트랜잭션에 포함된다. 그래서 기존 트랜잭션에 포함되었을 때는 기존 트랜잭션의 커밋이 완료될 때 커밋되고, 롤백 시에는 함께 롤백된다.    

### 4.2 REQUIRES_NEW

트랜잭션의 존재 여부와 상관없이 무조건 새로운 트랜잭션을 생성한다.  
기존 트랜잭션이 존재한다면 기존 트랜잭션을 보류하고 새로운 트랜잭션을 생성한다.  

### 4.3 SUPPORTS

기존 트랜잭션이 없으면 그대로 트랜잭션 없이 진행한다.  
기존 트랜잭션이 있으면 해당 트랜잭션에 포함된다.  

### 4.4 NOT_SUPPORTED

기존 트랜잭션이 없어도 새로운 트랜잭션을 생성하지 않는다.  
만약 기존 트랜잭션이 존재한다면 기존 트랜잭션을 보류하고, 트랜잭션 없이 진행한다.  

### 4.5 MANDATORY

기존 트랜잭션이 무조건 존재해야 하며, 해당 트랜잭션 하에서만 수행이 가능하다.  
만약 기존 트랜잭션이 없다면 `IllegalTransactionStateException`이 발생한다.   

### 4.6 NESTED

기존 트랜잭션이 없으면 새로운 트랜잭션을 만들고, 기존 트랜잭션이 있으면 중첩 트랜잭션을 만든다.  
커밋 시점은 기존 트랜잭션이 완료될 때이지만, 롤백 시엔 기존 트랜잭션에 영향을 주지 않는다.  
만약, 부모 트랜잭션이 롤백되며 해당 트랜잭션도 함께 롤백된다.   

### 4.7 NEVER

어떤 경우에도 트랜잭션을 생성하지 않는다.

---

위에서 설명한 전파 레벨을 기존 트랜잭션의 여부에 따라 간략히 확인해보자.

||기존 트랜잭션 O|기존 트랜잭션 X|
|---|---|---|
|REQUIRED|기존 트랜잭션에 참여|새롭게 생성|
|REQUIRES_NEW|새롭게 생성|새롭게 생성|
|SUPPORTS|기존 트랜잭션에 참여|트랜잭션 없이 진행|
|NOT_SUPPORTED|트랜잭션 없이 진행|트랜잭션 없이 진행|
|MANDATORY|기존 트랜잭션에 참여 |예외 발생|
|NESTED|중첩(자식) 트랜잭션 생성|새롭게 생성|
|NEVER|예외 발생|트랜잭션 없이 진행|

**출처**  
[Real MySQL8.0 1편](https://www.yes24.com/Product/Goods/103415627)  
https://kth990303.tistory.com/385

<script src="https://giscus.app/client.js"
        data-repo="j-jeongeun/github.io.comments"
        data-repo-id="R_kgDOJIg9UQ"
        data-category="Announcements"
        data-category-id="DIC_kwDOJIg9Uc4CVz67"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="ko"
        crossorigin="anonymous"
        async>
</script>