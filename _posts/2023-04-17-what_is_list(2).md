---
title: Collection - List 맛보기(수행속도 비교)
date: 2023-04-17
categories: [List]
tags: [Collection, List, ArrayList, LinkedList]
---
바로 앞 포스팅에서는 Collection 중, ArrayList와 LinkedList의 내부 구조를 뜯어보았다.  
이번 글에서는 ArrayList와 LinkedList의 수행 속도를 비교해 볼 것이다. 

## 1. add() 실행 속도 비교

데이터를 추가하는 경우도 2가지로 나누어 비교해 보았다.

1. 순차적으로 데이터를 추가할 경우
2. 중간에 데이터를 추가할 경우

### 1-1. 순차적으로 데이터를 추가할 경우

`설명하기 앞서 해당 테스트 코드는 정확한 속도 비교를 위해 nanoTime()으로 측정되었다.`

```java
public void speedTestCompareToArrayListAndLinkedListInOrderAdd() {

    /**
    * list에 1000개의 데이터를 순차적으로 추가
    */
    long timeOfArrayList = addedInOrderArrayList();

    long timeOfLinkedList = addedInOrderLinkedList();

    log.info("arrayList 수행 속도 {}", timeOfArrayList);
    log.info("linkedList 수행 속도 {}", timeOfLinkedList);
}

private long addedInOrderArrayList() {
    ArrayList<Integer> list = new ArrayList<>();

    long startTime = System.nanoTime();

    for (int i=1; i<=1000; i++) {
        list.add(i);
    }

    long endTime = System.nanoTime();

    return endTime - startTime;
}

private long addedInOrderLinkedList() {
    LinkedList<Integer> list = new LinkedList<>();

    long startTime = System.nanoTime();

    for (int i=1; i<=1000; i++) {
        list.add(i);
    }

    long endTime = System.nanoTime();

    return endTime - startTime;
}
```

![image](https://user-images.githubusercontent.com/121920173/232840591-7e2f214e-5691-41c2-bb81-cabf5dc830b4.png)

결과를 통해 알 수 있듯이 순차적으로 데이터를 추가할 경우 ArrayList가 LinkedList보다 4배 이상 빠르다.  
하지만 나노초로 수행한 결과이기 때문에 초로 변환 할 경우, 0.0000377초와 0.0001722초 변환되고 0.0001초 정도의 차이라고 생각할 수 있다.

### 1-2. 중간에 데이터를 추가할 경우

그렇다면 중간에 데이터를 추가할 경우 어떤 결과가 나올까?

```java
public void speedTestCompareToArrayListAndLinkedListInTheMiddleAdd() {

    /**
    * 10000개의 데이터가 들어있는 list에 중간 인덱스에 10개의 데이터 추가
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

10000개의 데이터가 있는 list에서는 10개의 데이터를 중간에 추가할 때는 ArrayList와 LinkedList의 속도 차이가 크게 나지 않아 데이터의 크기와 중간에 삽입되는 데이터의 갯수를 늘려 보았다.  
데이터의 전체 크기 그리고 중간에 삽입되는 데이터가 늘어날수록 ArrayList와 LinkedList의 수행 속도 차이가 확연히 난다는 걸 확인할 수 있다.

순차적으로 추가되는 데이터의 경우 ArrayList가 중간에 데이터가 추가될 경우 LinkedList의 속도가 빠르다는 결론을 얻을 수 있다. (0.0014599초와 0.0002856초)

<span style="color:red;">
**- 주의사항**  
해당 테스트 코드는 편의를 위해 데이터의 전체 크기 중 앞쪽 인덱스 부분에 데이터를 중간 삽입하였다.  
하지만, 해당 테스트 이외의 추가적인 테스트 코드를 작성 후 확인 해 본 결과,  
앞서 내린 **중간 인덱스에 값을 추가할 때는 LinkedList의 수행속도가 빠르다**라는 결론과 반대되는 결과를 확인할 수 있었다.  
해당 결과와 그 이유에 대해서는 다음 글에서 소개할 예정이니 참고하길 바란다.      
</span>

👉🏻 [당신이 알던 LinkedList가 아니다.](https://j-jeongeun.github.io/posts/what_is_list(3)/)

## 2. remove() 실행 속도 비교

데이터를 삭제하는 경우도 2가지로 나누어 비교해 보았다.

1. 순차적으로 데이터를 삭제할 경우
2. 중간에 데이터를 삭제할 경우

### 2-1. 순차적으로 데이터를 삭제할 경우

```java
public void speedTestCompareToArrayListAndLinkedListInOrderRemove() {  
  
    /**  
    * list에 50000개의 데이터를 순차적으로 삭제  
    */  
    long timeOfArrayList = removedInOrderArrayList();  

    long timeOfLinkedList = removedInOrderLinkedList();  
  
    log.info("arrayList 수행 속도 {}", timeOfArrayList);  
    log.info("linkedList 수행 속도 {}", timeOfLinkedList);  
}
```

![image](https://user-images.githubusercontent.com/121920173/234609962-3da40258-2da6-4102-bf32-c4cae3e42651.png)


50000개 이상의 데이터를 순차적으로 삭제할 경우(해당 테스트에서는 마지막 인덱스부터 순서대로 삭제하였다.) ArrayList와 LinkedList의 수행 속도 차이가 약 2배 정도 난다는 것을 확인할 수 있다. (0.0012989초와  0.0024526초)

### 2-2. 중간에 데이터를 삭제할 경우

```java
public void speedTestCompareToArrayListAndLinkedListInOrderRemove() {  
  
    /**  
    * list에 50000개의 데이터를 순차적으로 삭제  
    */  
    long timeOfArrayList = removedInOrderArrayList();  

    long timeOfLinkedList = removedInOrderLinkedList();  
  
    log.info("arrayList 수행 속도 {}", timeOfArrayList);  
    log.info("linkedList 수행 속도 {}", timeOfLinkedList);  
}
```

![image](https://user-images.githubusercontent.com/121920173/234610866-f1ab6b0f-2d31-4d40-a6f3-e80f3ca004ae.png)

5000개의 데이터가 있는 list에서는 중간의 랜덤 인덱스 값 10개를 삭제할 때는 LinkedList가 ArrayList보다 빠르다는 것을 확인할 수 있다. (0.000025초와 0.0000189초)

순차적으로 삭제되는 데이터의 경우 ArrayList가 중간에 데이터가 삭제될 경우 LinkedList의 속도가 빠르다는 결론을 얻을 수 있다.

## 3. 결론

**순차적으로 데이터를 추가/삭제 할 때는 ArrayList가 List의 중간에 데이터를 추가/삭제 할 때는 LinkedList가 조회를 할때는 ArrayList의 속도가 빠르다는 것을 확인할 수 있었다.**

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