---
title: 동시성 처리는 어떻게 하는게 좋을까?
date: 2023-08-04
categories: [동시성]
tags: [동시성, synchronized, 낙관적 락, 비관적 락]
---

타임딜을 주제로 하는 개인 프로젝트 진행 중에 동시성 처리에 대한 공부가 필요해서, 이번에 동시성 처리에 대한 여러가지 해결 방법에 대해 알아보기로 하였다.

## 1. 문제점

우선, 내 프로젝트에서는 일정 기간 동안 할인된 가격으로 상품을 주문할 수 있는데,
주문자가 한꺼번에 몰리는 상황에서 재고의 갯수가 정확하게 지켜지고 있는지가 문제였다.

일반적으로는 당연히 지켜지고 있겠지라고 생각하지만, 실제로 요청이 한꺼번에 몰릴 때는 문제가 발생한다.

서로 다른 Thread에서 재고(STOCK)에 접근할 때, 이미 다른 Thread가 선점 중이라며 `race condition(경쟁상태)`가 된다. 서로 다른 Thread에서 같은 메모리를 공유하면서 원하지 않는 결과가 나오고 이를 위해 우리는 `동시성 처리`를 필수로 해주어야 한다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/70a2b44c-ea9c-4102-ac3f-1e42a4536c09)

## 2. 해결방안

### 2.1 synchronized

코드 레벨에서 생각할 수 있는 방법 중 하나이다.  
`synchronized`는 메소드명 앞 또는 메소드 내 `synchronized  블록`으로 선언할 수 있다.

```java
example)
public synchronized int plus(int n) {...}

public int minus(int n) {
	synchronized{...}
}
```

`synchronized`을 사용할 때 가장 중요하게 생각해야 하는 점은 오직 하나의 쓰레드만 동작한다는 것이다.  
`synchronized`가 선언된 plus() 메소드를 동시에 100번 요청한다면 첫번째 요청이 완료된 후, 두번째, 세번째 요청이 실행된다는 것이다.  
글로만 봤을 때는 오직 하나의 쓰레드만 동작하면 동시성 처리가 완벽한게 아닌가라고 생각할 수도 있지만, 만약 해당 메소드의 코드가 100줄이거나 복잡한 로직이 섞여 있다면 해당 메소드를 1번 요청 시, 몇 초 혹은 그 이상이 소요될 수도 있다. 만약 1분이 걸린다면 100번째의 요청이 완료될려면 최소 100분은 기다려야 하는 것이다.  
이렇게 시간이 늘어지게 되면 과연 user들이 이 서비스를 사용할까?  
그래서 `synchronized`을 사용한다면 메소드 앞이 아닌 메소드 내 `synchronized 블록`으로 사용하는 걸 추천한다.

그렇다면, 동시성 처리가 필요한 로직만 `synchronized 블록`을 사용하면 되지 않을까?

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/f56bcc9f-4de6-4e35-8609-ca5137d95fa7)

`synchronized`를 사용할 때의 문제는 2대 이상의 서버를 사용할 때이다.  
 하나의 Process내에서 `synchronized`는 Thread-safe 하지만, 여러 대의 서버에서는 Thread-unsafe하다.
 서로 다른 서버의 Thread의 sync를 맞출 수 없기 때문이다.
 
단일 서버 기준에서는 문제가 없을 수 있지만, 서버가 여러 대 일 경우에는 `synchronized`는 동시성 처리를 위한 정확한 답은 아니다.

**synchronized + @Transactional**
또한, 단일 서버 내라도 메소드내에서 insert/update 쿼리가 여러개 존재한다면 예외가 발생했을때, 해당 데이터를 rollback 처리하기 어렵다.  
데이터 rollback을 가장 쉽게 처리할 수 있는 방법은 @Transactional이 있지만, 해당 애노테이션을 사용하면 동시성을 보장할 수 없다.  
왜일까?

```java
@Transactional
public synchronized int plus(int n) {...}
```

해당 코드의 실제 실행 순서는 아래와 같다고 생각하면 쉽다.
1) `transaction.start();`
2) `service.plus();`
3) `transaction.commit();`

실제 서비스 로직을 실행한 다음, 변경된 데이터가 `commit`되기 전에 새로운 transaction이 생성되어 원하는 결과를 얻지 못할 수도 있기 때문인다.

위에서 알아본 결과들로 `synchronized`는 동시성 처리를 위한 완벽한 방법은 아닌거 같다.  
동시성 처리를 위한 또 다른 방법을 알아보자.  

### 2.2 DB Lock

DB에서도 동시성 처리를 위한 방법을 제공한다.  
많이들 들어보았겠지만 낙관적 락 / 비관적 락 등이 여기에 해당된다.

#### 2.2.1 낙관적 락(Optimistic Lock)

- DB에 직접적인 Lock을 걸어 제어하는 것인 아닌 Version 관리를 통해 동시성 처리를 해결한다.
- 로직이 실패했을 때, rollback 처리를 수기로 해주어야 한다.

#### 2.2.2 비관적 락(Pessimistic Lock)

- 비관적 락은 실제 DB 데이터에 Lock을 걸어 접근할 수 없도록 막는 방법이다.
- 일반적으로 SELECT 문에 많이 사용된다. (`SELECT ... FOR UPDATE`)
- `SELECT ... FOR UPDATE` 쿼리가 실행될 때, `갭락`이 발생하는데 이 때 새로운 데이터는 INSERT 할 수 없다.- (갭락 : 레코드와 레코드 사이의 간격에 새로운 레코드가 생성(INSERT) 되는 것을 제어한다. )
- Lock을 가진 쓰레드가 종료되기 전까지 해당 데이터에 접근할 수 없다.

#### 2.2.3 낙관적 락(Optimistic Lock) VS 비관적 락(Pessimistic Lock)

- 성능적인 면에서는 낙관적 락이 좋지만, 낙관적 락에 가장 큰 단점은 예외 발생 시, rollback 처리가 어렵다는 것이다. 예외가 발생하지 않는다면 낙관적 락을 사용하면 좋겠지만 **예외 발생에 예외는 없다.**

### 2.3 redis

DB의 Lock을 이용하는 방법 이외에도 redis를 이용해서도 동시성 문제를 해결할 수 있다.
하지만 이번 포스팅에서는 간단하게만 설명하고 redis를 이용한 동시성 처리에 대해서는 생략하도록 하겠다.  
(+ redis는 사용 예정이라 아마 조만간 포스팅 될 수 있다.😄)

- redis를 이용한 동시성 처리
	- redis는 Single Thread로 여러 Thread가 동시에 같은 자원을 공유할 일이 없다.
	- 싱글스레드로 동작하기 때문에 데이터가 atomic하게 유지되어 동시성 이슈를 해결할 수 있다.
	- in-memory로 속도도 빠르다.

## 3. 각 각의 해결방안에 대한 부하테스트

각 해결 방법에 대한 결과를 수치로 확인하기 위해 nGrinder를 사용해보았다.  
nGrinder는 이번에 설치하여 처음 사용해 보았는데, 설치하면서 발생한 몇가지 오류에 대해 혹시 몰라 추가로 작성해보았다. 

**nGrinder를 이용한 테스트**
동시에 다건의 요청이 몰렸을 시(해당 요청 건수는 서비스의 성격에 따라 다를 수 있음), 발생할 수 있는 문제점을 확인 & test할 수 있다.
 어느 수 이상의 요청이 몰렸을 시, 병목 현상이 발생하거나 서버가 죽는 부분을 확인하여 서비스 오픈 전 사전에 해결할 수 있다. 

<details>
<summary>nGrinder 테스트 오류</summary>
<div markdown="1">

nGrinder 설치 후, agent를 실행하여 google.com을 요청하는 테스트를 먼저 진행하였다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/f022c1e2-7784-46c7-9d00-1053ad1e4815)

근데,  아래와 같은 오류가 발생하였다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/af475efb-fa7b-4d7e-a2dd-57aeb3bb4b7c)

해당 오류 내역을 확인해보니 현재 jdk17을 사용중인데, 해당 버전으로 compile된 class파일을 읽을 수 없다는 내용이었다.

알고 보니, 현재 nGrinder는 jdk11까지 지원중이라고 한다....

예상치 못한 변수다.

jdk11을 추가 설치하여 다시 실행해보았다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/d6bfd3e6-89a5-4235-84f7-2c4884232600)

정상적으로 수행되는 것을 확인할 수 있었다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/22519ecc-26da-4932-9f13-464a72392208)

</div>
</details>

<br>
동시성 처리에서 발생하는 문제를 확인하기 위한 테스트임으로, 로그인과 같은 부가 기능은 제외하고 `주문처리 로직`에 대해서만 테스트를 진행하기로 하였다.  
아래의 조건과 같이 100명의 가상 User를 두고 테스트를 진행하였다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/6a8317e8-5275-457b-835d-aac73b67f76a)

### 3.1 동시성 처리 적용 전

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/87aceeb9-977c-4d3a-8bb0-03e73c9beed2)

성공한 케이스보다 에러수가 월등히 많다.  
동시서 처리가 되지 않은 케이스이기 때문에 데드락이 발생하면서 너무 많은 에러가 발생하였다.  
에러 발생률이 너무 높아 도중에 중지되었다.

<span style="color:red; font-weight:bold;">실패!😥</span>

### 3.2 synchronized 적용

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/50af7e04-d052-406c-971b-4d3312a25dde)

내가 작성한 주문 로직에는 INSERT/UPDATE 되기 전 예외 처리가 되어 있었기 때문에 정상적으로 수행되었다. Erros수가 0인 것을 확인할 수 있다.

TPS(Test Per Seconds) : 4.8  
Mean Test Time : 10,129ms  
Executed Tests : 125

<span style="color:red; font-weight:bold;">성공!😗</span>

### 3.3 비관적락(Pessimistic Lock) 적용

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/db4ad3eb-3833-4264-9ae3-c1fcb86e9498)

비관적 락을 이용한 방법에서도 Error 발생 없이 수행되었다. 동일 시간 동안 `synchronized`보다 많은 수의 테스트를 수행한 것을 확인할 수 있다.

TPS(Test Per Seconds) : 7.1  
Mean Test Time : 10,667ms  
Executed Tests : 185

<span style="color:red; font-weight:bold;">성공!😊</span>

## 4. 결론

동시성 처리를 할 수 있는 방법에 대해서 이번에 공부해보았다.  
생각보다 이해하는데 다른 개념들보다 오래 걸렸던 거 같다.  

nGrinder를 이용한 테스트를 통해 성능적인 면에서 `비관적락(Pessimistic Lock)` > `synchronized`인 것을 확인하였으나, `비관적락(Pessimistic Lock)`을 이용한 동시성 처리도 성능면에서 좋아 보이지는 않는다.🥲  
이번에는 동시성 처리에 대한 기본적인 개념과 방법에 대해 알아 보았으니, 다음은 좀 더 나은 방법으로 처리할 수 있는 방법들에 대해서 공부해봐야겠다.  
(+ redis나 메시지 큐를 이용하여 좀 더 성능을 높일 수 있을 것으로 예상되는데, 다음 번에 변경해서 결과를 비교해야겠다라는 숙제를 남겼다.)

**참고 출처**  
https://digda.tistory.com/41

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