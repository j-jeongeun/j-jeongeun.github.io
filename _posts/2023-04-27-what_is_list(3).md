---
title: LinkedList가 과연 ArrayList보다 빠를까?
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
크게 3가지 경우로 나누어 테스트 코드를 작성해보았다.

1. 맨 앞 인덱스에 값을 추가한 경우
2. 중간 인덱스에 값을 추가한 경우
3. 맨 뒤 인덱스에 값을 추가한 경우

3가지의 경우를 이제 차례대로 알아보자.

## 1. 맨 앞 인덱스에 값을 추가한 경우

해당 테스트 코드는 전체 50,000개의 데이터 중 맨 앞에 1,000개의 데이터를 추가하였다.

```java
public void 수행_속도_비교_ArrayList_vs_LinkedList() {  
  
    /**  
    * 50,000개의 데이터가 들어있는 list에 중간 인덱스에 1,000개의 데이터 추가  
    */  
    addedInTheMiddleArrayList();  
    addedInTheMiddleLinkedList();  
}  
  
private void addedInTheMiddleArrayList() {  
    ArrayList<Integer> list = new ArrayList<>();  

    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    StopWatch stopWatch = new StopWatch("timeOfArrayList");  
  
    stopWatch.start();  
  
    for (int i=0; i<1000; i++) {  
        list.add(0, 0);  
    }  
  
    stopWatch.stop();  
  
    System.out.println(stopWatch.shortSummary());  
}  
  
private void addedInTheMiddleLinkedList() {  
    LinkedList<Integer> list = new LinkedList<>();  
  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    StopWatch stopWatch = new StopWatch("timeOfLinkedList");  
  
    stopWatch.start();  
  
    for (int i=0; i<1000; i++) {  
        list.add(0, 0);  
    }  
  
    stopWatch.stop();  
  
    System.out.println(stopWatch.shortSummary());  
}
```

50,000개의 크기 이외에도 10,000개/100,000개까지 추가로 테스트 해보았다.

### 1-1. 10,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235293835-095977ba-0f70-4deb-a60d-66c5a03a99a2.png)

### 1-2. 50,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235293943-44649ddc-c24d-402d-a810-ac29d6012585.png)

### 1-3. 100,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235293998-80771f9b-7a95-4d98-8d5d-7bc865907693.png)

데이터의 전체 크기에 따른 수행 속도를 확인하기 위해 중간에 삽입되는 데이터는 1,000개로 동일하게 수행하였다. 

||ArrayList|LinkedList|
|---|---|---|
|10,000개|0.0020604초|**0.0001921초**|
|50,000개|0.0075174초|**0.0001936초**|
|100,000개|0.014768초|**0.0001133초**|

해당 테스트 코드에서는 LinkedList의 수행 속도가 압도적으로 빠른 것을 확인할 수 있었다.  
이 결과는 우리가 예상했던 결과와 일치한다.

그리고 데이터의 전체 크기가 늘어날 수록 ArrayList와 LinkedList의 수행 속도가 약 10배/40배/130배까지 차이가 나는 것을 확인할 수 있었다.

## 2. 중간 인덱스에 값을 추가한 경우

해당 테스트 코드는 전체 50,000개의 데이터 중 정 가운데 인덱스에서 1,000개의 데이터를 추가하였다.

```java
public void 수행_속도_비교_ArrayList_vs_LinkedList() {  
  
    /**  
    * 50,000개의 데이터가 들어있는 list에 중간 인덱스에 1,000개의 데이터 추가  
    */  
    addedInTheMiddleArrayList();  
    addedInTheMiddleLinkedList();  
}  
  
private void addedInTheMiddleArrayList() {  
    ArrayList<Integer> list = new ArrayList<>();  

    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    StopWatch stopWatch = new StopWatch("timeOfArrayList");  
  
    stopWatch.start();  
  
    for (int i=0; i<1000; i++) {  
        list.add(list.size()/2, 25000);  
    }  
  
    stopWatch.stop();  
  
    System.out.println(stopWatch.shortSummary());  
}  
  
private void addedInTheMiddleLinkedList() {  
    LinkedList<Integer> list = new LinkedList<>();  
  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    StopWatch stopWatch = new StopWatch("timeOfLinkedList");  
  
    stopWatch.start();  
  
    for (int i=0; i<1000; i++) {  
        list.add(list.size()/2, 25000);  
    }  
  
    stopWatch.stop();  
  
    System.out.println(stopWatch.shortSummary());  
}
```

### 2-1. 10,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235306328-c90c68e7-41d9-4e2d-96dd-a24c3f25ee6f.png)

### 2-2. 50,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235306397-f06373cd-781f-417e-8060-30dcf3a71332.png)

### 2-3. 100,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235306429-0f47c4a4-4c28-4912-ac98-05722e20f277.png)

데이터의 전체 크기에 따른 수행 속도를 확인하기 위해 중간에 삽입되는 데이터는 1,000개로 동일하게 수행하였다. 

||ArrayList|LinkedList|
|---|---|---|
|10,000개|**0.0010974초**|0.010826초|
|50,000개|**0.0038859초**|0.03905초|
|100,000개|**0.0071742초**|0.079298초|

해당 테스트 코드에서는 ArrayList의 수행 속도가 빠른 것을 확인할 수 있었다.  
이 결과는 우리가 예상했던 결과와 **반대**이다.

중간에 데이터가 추가될 경우 데이터의 전체 크기와 상관없이 수행 속도가 전체적으로 약 10배 정도 차이가 나는 것을 확인할 수 있었다.

## 3. 맨 뒤 인덱스에 값을 추가한 경우

해당 테스트 코드는 전체 50,000개의 데이터 중 맨 뒤 인덱스에 1,000개의 데이터를 추가하였다.

```java
public void 수행_속도_비교_ArrayList_vs_LinkedList() {  
  
    /**  
    * 50,000개의 데이터가 들어있는 list에 중간 인덱스에 1,000개의 데이터 추가  
    */  
    addedInTheMiddleArrayList();  
    addedInTheMiddleLinkedList();  
}  
  
private void addedInTheMiddleArrayList() {  
    ArrayList<Integer> list = new ArrayList<>();  

    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    StopWatch stopWatch = new StopWatch("timeOfArrayList");  
  
    stopWatch.start();  
  
    for (int i=0; i<1000; i++) {  
        list.add(list.size()-1, 50000);  
    }  
  
    stopWatch.stop();  
  
    System.out.println(stopWatch.shortSummary());  
}  
  
private void addedInTheMiddleLinkedList() {  
    LinkedList<Integer> list = new LinkedList<>();  
  
    for (int i=1; i<=50000; i++) {  
        list.add(i);  
    }  
  
    StopWatch stopWatch = new StopWatch("timeOfLinkedList");  
  
    stopWatch.start();  
  
    for (int i=0; i<1000; i++) {  
        list.add(list.size()-1, 50000);  
    }  
  
    stopWatch.stop();  
  
    System.out.println(stopWatch.shortSummary());  
}
```

### 3-1. 10,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235294640-be24c28d-021e-4f0b-bee7-4ab39fd8103b.png)

![image](https://user-images.githubusercontent.com/121920173/235305335-badb3ee7-721c-4349-92a3-b6189132b34e.png)

### 3-2. 50,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235294734-546876d5-43d9-435e-a639-513b4b1b23a5.png)

![image](https://user-images.githubusercontent.com/121920173/235305363-6d907e89-9c6d-4da5-b5ae-62862c3bb907.png)

### 3-3. 100,000개의 데이터의 1,000개의 데이터를 추가할 경우 - 결과
![image](https://user-images.githubusercontent.com/121920173/235294777-f572c163-3d6a-4452-b1df-cf8ca0483e91.png)

데이터의 전체 크기에 따른 수행 속도를 확인하기 위해 중간에 삽입되는 데이터는 1,000개로 동일하게 수행하였다.

해당 테스트 코드에서는 10,000개/ 50,000개 리스트의 마지막에 데이터를 추가할 경우 어떨 때는 ArrayList가 어떨 때는 LinkedList가 빠르게 나왔다.

하지만 100,000개의 데이터에서는 ArrayList가 LinkedList보다 약 2.5배 빠른 것을 확인할 수 있었다.

위 결과로 알 수 있는 내용은 `순차적으로 값을 추가할 경우는 LinkedList보다 ArrayList가 빠르다`라고 알고 있지만 50,000개의 데이터까지는 ArrayList와 LinkedList 중 정확히 어떤 것이 수행 속도의 우의에 있는지는 확인하기 어렵다는 것이다.

50,000개 까지는 수행속도의 큰 차이는 나지 않지만 그 이상의 크기부터 ArrayList의 수행속도가 빠르다는 결과를 얻을 수 있다.

## 4. 왜일까?

그렇다면 도대체 왜 이런 결과가 나오는 걸까?  
앞서 포스팅에서 LinkedList의 가장 중요한 부분이 빠져있었다.

ArrayList의 중간 인덱스에 데이터가 추가되었을 경우는 `System.arraycopy()` 메소드를 이용하여 배열을 복사한다.

![image](https://user-images.githubusercontent.com/121920173/234859044-a3768c7c-0a83-4229-89d0-13a2d39dd585.png)

LinkedList의 경우, 복사하기 이전에 **해당 인덱스의 값에 접근하기 위해 인덱스 값에 따라 인덱스의 맨 앞 또는 맨 뒤부터 접근하게 된다.**  
만약 50,000개의 데이터 중, 25,000번째의 인덱스에 값을 추가할 경우 49,999~25,000번까지 인덱스를 탐색하게 된다.  
데이터를 추가하기 전 해당 인덱스를 찾기 위한 탐색과정이 있기 때문에 우리가 예상했던 것보다 LinkedList의 수행속도가 느려진다.  
아래 코드의 if절과 for문을 확인해보면 해당 인덱스의 위치가 전체 크기의 중간에 위치할 수록 접근해야 하는 인덱스 값들이 많기 때문에 느려지며, 초반과 후반부의 위치할때는 더 빨리 수행된다. 

![image](https://user-images.githubusercontent.com/121920173/234890638-22264cf1-6e81-4756-961f-c110babfa1ae.png)

## 5. 결론

||10,000개|50,000개|100,000개|
|---|---|---|---|
|ArrayList(맨앞)|0.0020604초|0.0075174초|0.014768초|
|**LinkedList(맨앞)**|**0.0001921초**|**0.0001936초**|**0.0001133초**|
||
|**ArrayList(중간)**|**0.0010974초**|**0.0038859초**|**0.0071742초**|
|LinkedList(중간)|0.010826초|0.03905초|0.079298초|
|
|ArrayList(맨뒤)|||0.0001604초|
|LinkedList(맨뒤)|||**0.0004145초**|

위에서 나온 결과를 정리해 보았다.  
맨 뒤 인덱스에 값을 추가하는 경우는 100,000개의 결과만 정리하였다.
 
실제로 3가지 테스트 중, 예상된 결론과 맞아 떨어지는 결과는 
1. 맨 앞 인덱스에 데이터를 추가하는 경우
2. 일정 크기 이상의 리스트에서 맨 뒤에 데이터를 추가하는 경우

밖에 없었다.  
맨 뒤에 데이터를 추가한 경우에도 LinkedList가 빠르긴 하지만 50,000개 까지는 큰 차이가 없다는 것도 확인할 수 있었다.

정리하자면,  
1. **맨 앞 인덱스에 값을 추가하는 경우, 데이터의 크기가 커질수록 LinkedList의 수행속도가 ArrayList의 수행속도보다 빨랐다.**  
2. **중간 인덱스에 값을 추가하는 경우, 데이터의 크기와 상관없이 ArrayList의 수행속도가 LinkedList의 수행속도보다 약 10배 빨랐다. (최대 100,000개 기준)**  
3. **맨 뒤 인덱스에 값을 추가하는 경우, 일정 크기(50,000개) 이상일 때 LinkedList의 수행속도가 ArrayList의 수행속도보다 빨랐다.**

그렇다면 도대체 어떤 경우에 LinkedList를 사용해야 할까?  
우선 테스트 결과를 통해 명확하게 차이가 나는 부분은 맨 앞 인덱스에 데이터를 추가할 때이다.

하지만 우리는 데이터를 추가할 때 앞쪽에 넣을때는 LinkedList를 이용하고 중간에 넣을 때는 다시 ArrayList를 이용하는 경우는 없다.  
전체적인 데이터의 크기와 중간에 삽입되는 데이터가 있을 경우의 수 등을 생각하여 상황에 따라 적절하게 선택해서 사용하는 게 좋아 보인다.

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