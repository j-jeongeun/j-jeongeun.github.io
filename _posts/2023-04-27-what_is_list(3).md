---
title: 당신이 알던 LinkedList가 아니다.
date: 2023-04-27
categories: [List]
tags: [Collection, List, ArrayList, LinkedList]
---
ArrayList와 LinkedList의 수행 속도를 비교하는 테스트 코드를 작성하면서 예상하지 못한 결과가 나와 당황했다.  
물론 어떤 결과가 나오겠지라는 예상을 하는 건 좋지 않지만 ArrayList와 LinkedList가 어떤 경우에 쓰여야 하고 어떨 때 빠를까?라는 이론은 익히 많은 블로그에 작성된 글이 많아 다들 알고 있을거라 생각한다.

그렇다면 예상하지 못한 결과라는 건 어떤 거였을까?  

바로 말하자면, 인덱스의 중간에 데이터를 추가하는 경우 보통 LinkedList가 ArrayList보다 수행 속도가 빠르다라고 이해하고 있다.  
하지만, 여러 테스트 코드를 작성 해본 결과 추가되는 인덱스의 위치에 따라 수행 속도의 차이가 났다.

앞서 포스팅에서 인덱스 중간에 데이터를 삽입할 경우 ArrayList보다 LinkedList가 훨씬 빠르다. 라고 말하였지만 **이제는 이 결론을 뒤집을 것이다.**

그렇다면 어떤 경우에 이런 결과가 나오고 왜 나오는 건지 다시 확인해보자.  
크게 3가지 경우로 나누어 추가적으로 테스트 코드를 작성해보았다.

1. 앞쪽 인덱스에 값을 추가한 경우
2. 중간 인덱스에 값을 추가한 경우
3. 뒤쪽 인덱스에 값을 추가한 경우

3가지의 경우를 이제 차례대로 알아보자.

## 1. 앞쪽 인덱스에 값을 추가한 경우

앞선 포스팅에서 작성했던 테스트 코드이다.  
해당 테스트 코드는 전체 50000개의 데이터 중 500번째에서 100개의 데이터를 추가하였다.

```java
public void speedTestCompareToArrayListAndLinkedListInTheMiddleAdd() {

    /**
    * 50000개의 데이터가 들어있는 list에 중간 인덱스에 100개의 데이터 추가
    */
    long timeOfArrayList = addedInTheMiddleArrayList();  
  
    long timeOfLinkedList = addedInTheMiddleLinkedList();

    log.info("arrayList 수행 속도 {}", timeOfArrayList);
    log.info("linkedList 수행 속도 {}", timeOfLinkedList);
}

private long addedInTheMiddleArrayList() {
    ArrayList<Integer> list = new ArrayList<>();  
	  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    long startTime = System.nanoTime();  
  
    for (int i=0; i<100; i++) {  
        list.add(500, 500);  
    }  

    long endTime = System.nanoTime();  
 
    return endTime - startTime;
}

private long addedInTheMiddleLinkedList() {
    LinkedList<Integer> list = new LinkedList<>();  
	  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    long startTime = System.nanoTime();  
	  
    for (int i=0; i<100; i++) {  
        list.add(500, 500);  
    }  
  
    long endTime = System.nanoTime();  
  
    return endTime - startTime;
}
```

![image](https://user-images.githubusercontent.com/121920173/234621389-44bb004c-df53-4ca7-a618-6f8af2cccaaf.png)

해당 테스트 코드에서는 LinkedList의 수행 속도가 압도적으로 빠른 것을 확인할 수 있었다. (결과(s) : 0.0014599초와 0.0002856초)  
이 결과는 우리가 예상했던 결과와 일치한다.

## 2. 중간 인덱스에 값을 추가한 경우

해당 테스트 코드는 전체 50000개의 데이터 중 25000번째에서 100개의 데이터를 추가하였다.

```java
public void speedTestCompareToArrayListAndLinkedListInTheMiddleAdd() {

    /**
    * 50000개의 데이터가 들어있는 list에 중간 인덱스에 100개의 데이터 추가
    */
    long timeOfArrayList = addedInTheMiddleArrayList();  
  
    long timeOfLinkedList = addedInTheMiddleLinkedList();

    log.info("arrayList 수행 속도 {}", timeOfArrayList);
    log.info("linkedList 수행 속도 {}", timeOfLinkedList);
}

private long addedInTheMiddleArrayList() {
    ArrayList<Integer> list = new ArrayList<>();  
	  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    long startTime = System.nanoTime();  
  
    for (int i=0; i<100; i++) {  
        list.add(25000, 500);  
    }  

    long endTime = System.nanoTime();  
 
    return endTime - startTime;
}

private long addedInTheMiddleLinkedList() {
    LinkedList<Integer> list = new LinkedList<>();  
	  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    long startTime = System.nanoTime();  
	  
    for (int i=0; i<100; i++) {  
        list.add(25000, 500);  
    }  
  
    long endTime = System.nanoTime();  
  
    return endTime - startTime;
}
```

![image](https://user-images.githubusercontent.com/121920173/234854438-b2c65800-c38b-474b-b59b-1533acf0fb9c.png)

해당 테스트 코드에서는 ArrayList의 수행 속도가 압도적으로 빠른 것을 확인할 수 있었다. (결과(s) : 0.0007774초와 0.0053892초)  
이 결과는 우리가 예상했던 결과와 반대이다.

## 3. 뒤쪽 인덱스에 값을 추가한 경우

해당 테스트 코드는 전체 50000개의 데이터 중 40000번째에서 100개의 데이터를 추가하였다.

```java
public void speedTestCompareToArrayListAndLinkedListInTheMiddleAdd() {

    /**
    * 50000개의 데이터가 들어있는 list에 중간 인덱스에 100개의 데이터 추가
    */
    long timeOfArrayList = addedInTheMiddleArrayList();  
  
    long timeOfLinkedList = addedInTheMiddleLinkedList();

    log.info("arrayList 수행 속도 {}", timeOfArrayList);
    log.info("linkedList 수행 속도 {}", timeOfLinkedList);
}

private long addedInTheMiddleArrayList() {
    ArrayList<Integer> list = new ArrayList<>();  
	  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    long startTime = System.nanoTime();  
  
    for (int i=0; i<100; i++) {  
        list.add(40000, 500);  
    }  

    long endTime = System.nanoTime();  
 
    return endTime - startTime;
}

private long addedInTheMiddleLinkedList() {
    LinkedList<Integer> list = new LinkedList<>();  
	  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    long startTime = System.nanoTime();  
	  
    for (int i=0; i<100; i++) {  
        list.add(40000, 500);  
    }  
  
    long endTime = System.nanoTime();  
  
    return endTime - startTime;
}
```

![image](https://user-images.githubusercontent.com/121920173/234854775-3b8199f7-5c74-4279-beb8-6cb6ecc84070.png)

해당 테스트 코드 또한 ArrayList의 수행 속도가 빠른 것을 확인할 수 있었다. (결과(s) : 0.0004812초와 0.002466초)  
이 결과도 우리가 예상했던 결과와 반대이다.

## 4. 왜일까?

그렇다면 도대체 왜 이런 결과가 나오는 걸까?  
앞서 포스팅에서 LinkedList의 가장 중요한 부분이 빠져있었다.

ArrayList의 중간 인덱스에 데이터가 추가되었을 경우는 `System.arraycopy()` 메소드를 이용하여 배열을 복사한다.

![image](https://user-images.githubusercontent.com/121920173/234859044-a3768c7c-0a83-4229-89d0-13a2d39dd585.png)

LinkedList의 경우, 복사하기 이전에 **해당 인덱스의 값에 접근하기 위해 인덱스 값에 따라 인덱스의 맨 앞 또는 맨 뒤부터 접근하게 된다.**  
만약 50000개의 데이터 중, 25000번째의 인덱스에 값을 추가할 경우 49999~25000번까지 인덱스를 탐색하게 된다.  
데이터를 추가하기 전 해당 인덱스를 찾기 위한 탐색과정이 있기 때문에 우리가 예상했던 것보다 LinkedList의 수행속도가 느려진다.  
아래 코드의 if절과 for문을 확인해보면 해당 인덱스의 위치가 전체 크기의 중간에 위치할 수록 접근해야 하는 인덱스 값들이 많기 때문에 느려지며, 초반과 후반부의 위치할때는 더 빨리 수행된다. 

![image](https://user-images.githubusercontent.com/121920173/234890638-22264cf1-6e81-4756-961f-c110babfa1ae.png)

## 5. 결론

중간 인덱스에 값을 추가할 경우 LinkedList의 수행 속도가 빠르다라고 모두들 알고 있을 것이다.  
나도 이전 포스팅에서는 위와 같은 결론을 내렸다.  
하지만 테스트의 방법을 달리 하다보니 예상과는 다른 결과를 확인하여 이번 글을 작성하게 되었다.
 
실제로 3가지 테스트 중, 예상된 결론과 맞아 떨어지는 결과는 앞쪽 인덱스에 데이터를 추가한 경우밖에 없었다.  
중간 인덱스에 값을 추가할 경우 수행속도가 가장 많이 차이가 났으면 뒤쪽 인덱스에 값을 추가하였을 경우에도 차이가 나는 것을 확인할 수 있다.

그렇다면 도대체 어떤 경우에 LinkedList를 사용해야 할까?  
우선 테스트 결과를 통해 명확하게 차이가 나는 부분은 전체 크기의 비교적 앞부분에 데이터를 추가할 때는 LinkedList가 ArrayList보다 훨씬 뛰어나다는 것이다.  
하지만 우리는 데이터를 추가할 때 앞쪽에 넣을때는 LinkedList를 이용하고 뒤쪽에 넣을 때는 다시 ArrayList를 이용하는 경우는 없다.  
전체적인 데이터의 양과 중간에 삽입되는 데이터가 있을 경우의 수 등을 생각하여 상황에 따라 적절하게 선택해서 사용하는게 좋아보인다.

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