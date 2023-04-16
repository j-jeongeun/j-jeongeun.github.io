---
title: Collection - List 물고 뜯고 맛보기
date: 2023-04-11
categories: [List]
tags: [Collection, List, ArrayList, LinkedList]
---
Collection의 개념을 재정의 하는 시간을 가지면서, 기본적인 개념은 알겠으나 List 인터페이스의 구현체인 ArrayList와 LinkedList를 언제 어떤 목적으로 써야하는지에 대해 명확한 기준을 세우기 위해 ArrayList와 LinkedList의 내부 구조를 뜯어보기로 하였다. 

## 1. 우린 List야
 
우선 ArrayList와 LinkedList가 가지는 List의 공통적인 속성에 대해서 알아보자.

1. List는 순서를 가지고 있다.
2. add(), addAll(), remove(), removeAll(), contains() 등의 Collection 인터페이스에서 구현한 메소드를 가지고 있다.
3. get()의 경우 List 인터페이스에서 구현한 메소드이다.
4. 크기를 나타내는 메소드는 size()이다.

<hr>

이번 포스팅에서 중요하게 알아볼 내용은 ArrayList와 LinkedList의 데이터 추가(add), 삭제(delete), 조회(get)가 일어났을 때 객체 내부에서 일어나는 코드를 뜯어볼 것이다.

## 2. add() - ArrayList

먼저 ArrayList의 add() 메소드 구현방식을 알아보자.

우선,

<img width="334" alt="image" src="https://user-images.githubusercontent.com/121920173/230782296-3470b663-bc09-49b9-887a-02565a29c49f.png">

<img width="434" alt="image" src="https://user-images.githubusercontent.com/121920173/230782138-073603a4-df34-446a-9630-d20e962909cf.png">

list라는 ArrayList 객체를 생성하였을 때 ArrayList 클래스의 생성자를 확인 해본 결과,
`Constructs an empty list with an initial capacity of ten.`
이라는 문구를 확인할 수 있다.  
기본 생성자를 이용하여 생성하였을 때는 저장 공간을 기본 10으로 가진다.

그렇다면 기본 공간인 10을 기준으로 10개 이하의 데이터를 넣었을 때와 10개를 초과하였을 때의 데이터 추가 방식을 확인해보자.

### 2.1 크기가 10 이하인 경우

<img width="360" alt="image" src="https://user-images.githubusercontent.com/121920173/230782434-b7437ea6-3011-4074-9401-e703e01a52a5.png">

반복문이 수행된 이후, 6번째 인덱스에 값을 추가할 경우 어떤 일이 일어나는지 디버깅을 통해 확인해보자.

<table>
	<tr>
		<td><img width="600px" alt="image" src="https://user-images.githubusercontent.com/121920173/230782637-1b67fcae-9edb-466e-ad21-e8e61f981638.png"></td>
		<td><img width="300px" alt="image" src="https://user-images.githubusercontent.com/121920173/230782685-fbc7595b-3c36-4ee7-ae77-9e2785a31932.png"></td>
	</tr>
</table>

<table>
	<tr>
		<td><img width="700px" alt="image" src="https://user-images.githubusercontent.com/121920173/230782709-5e341d44-3d66-495a-8b77-59263900a224.png"></td>
		<td><img width="300px" alt="image" src="https://user-images.githubusercontent.com/121920173/230782774-db3f9048-da21-43f0-93ed-f8404978c675.png"></td>
	</tr>
</table>

순차적으로 실행되는 디버깅을 따라 변수의 값들도 같이 확인해본 결과, 기본 저장 공간인 10 이하에서 데이터를 추가할 경우 `add(E e, Object[] elementData, in s)` 메소드에서 if절은 패스하고 바로 다음 index값에 값을 넣어주고 size+1을 해준 다음 return true를 반환한다.

**여기서 처음 이해가 되지 않았던 부분은 if절의 조건문인 `s == elementData.length`의 값이 왜 false였을까?** 였다.

내가 생각한 `elementData.length`는 데이터의 크기값(size())인 5라고 생각을 했는데, 디버깅 값을 통해 확인해보면 
`elementData.length = 10`인 것을 확인할 수 있다.

<img width="384" alt="image" src="https://user-images.githubusercontent.com/121920173/230783234-a46885e7-5b8e-448b-8034-35805ff31a8f.png">

바로 전 단계의 디버깅 값의 elementData를 상세 확인해보면 5개의 데이터는 정상적으로 들어갔지만 `Not showing null elements`라는 문구가 보인다.

그렇다면 크기가 10을 초과한 list의 경우 해당 문구가 보이지 않을까?  
이것 또한 다음 `2.2 크기가 10을 초과한 경우`에서 알아보자.

그렇다면 다시 본론으로 돌아와서 `elementData.length = 10`은 왜 10일까?

<img width="434" alt="image" src="https://user-images.githubusercontent.com/121920173/230782138-073603a4-df34-446a-9630-d20e962909cf.png">

처음 ArrayList 객체를 생성할 때 초기 공간이 10으로 할당되었다는 것을 확인할 수 있다.

### 2.2 크기가 10을 초과한 경우

<img width="349" alt="image" src="https://user-images.githubusercontent.com/121920173/230783734-7e4155cd-96aa-4fc0-bad4-1e89d44a09fe.png">

list.size()의 값이 10보다 클 때, 데이터를 추가할 경우 어떤 일이 일어날까?

여기서 우리가 추가로 알아봐야 할 내용은 2.1에서 확인한 elementData의 `Not showing null elements`라는 문구가 동일하게 발생하냐이다.

<img width="326" alt="image" src="https://user-images.githubusercontent.com/121920173/230783805-10911ed7-8d9d-4781-9c9a-e66dd6ec79e1.png">

크기가 15인 list의 elementData를 상세히 보면 `Not showing null elements` 문구가 보이지 않는다.  
그렇다는 것은 크기가 10이하일 때는 list에서는 나머지 공간이 null로 채워져 있어 출력되지 않은 것이고, 크기가 10을 초과한 경우 기본 저장 공간을 초과하여서 해당 문구가 보이지 않는다.라는 결론을 내릴 수 있을 거 같다.

<img width="737" alt="image" src="https://user-images.githubusercontent.com/121920173/230783994-4e27b268-9276-49c2-b562-d35e75b15a84.png">

다시 돌아와서 add() 내부를 확인해보면 이번에는 if절에서는 true가 되어 `elementData = grow()`를 실행한다.

우선 다음 실행 순서를 설명하면서 순차적으로 이야기하겠다.

<img width="268" alt="image" src="https://user-images.githubusercontent.com/121920173/230784081-25365192-e968-4e8d-ba1d-64a2648a75aa.png">

grow() 메소드에서는 `현재크기 + 1`을 return한다.  
그 다음에는 int를 매개변수로 갖는 grow() 코드에서 배열을 아래와 같이 복사해준다.

<img width="486" alt="image" src="https://user-images.githubusercontent.com/121920173/230784419-0226c47e-8718-4cd0-a361-6bba2910cfc6.png">

다음 호출되는 `newCapacity(int minCapacity)` 메소드에서는 

<img width="662" alt="image" src="https://user-images.githubusercontent.com/121920173/230784509-3cd72127-52a0-4f8c-abe0-f101cea93935.png">

newCapacity = 22, minCapacity = 16으로 if절은 패스하고 return newCapacity를 반환한다.

여기서 또 알 수 있는 점은 list의 크기인 16이 아니라 `oldCapacity + (oldCapacity >> 1)`만큼 저장공간을 늘려준다는 것이다.

그리고 return된 공간에 elementData를 복사하여 return하고 저장공간 또한 늘어난다.

<img width="546" alt="image" src="https://user-images.githubusercontent.com/121920173/230785071-0e367321-b88d-4af2-b0e5-6a2ebfc34c7a.png">

이렇게 많은 과정을 통해 list에 값이 추가된다.  

### 2.3 값이 중간에 추가된 경우

![image](https://user-images.githubusercontent.com/121920173/231163531-5b648231-4039-40f8-96d9-bc655b93179c.png)

다음 과정을 간략히 설명해보면 배열을 복사하여 값을 재배치한다음 추가되는 인덱스의 값만 다시 재할당해준다.  
그리고  주석을 통해 알 수 있듯이 추가되는 위치의 인덱스 값 뒤의 값들은 오른쪽으로 한칸씩 옮겨진다고 생각하면 된다.

디버깅을 해보지 않았다면 이렇게 많은 과정을 거쳐야 하는지는 몰랐을 것이다.  
역시 코드는 뜯고 맛보아야 알 수 있다는 것을 한번 더 알게 해주는 경험이었다.

우리는 이제 ArrayList의 add() 메소드 하나만 뜯어보았다. 이와 비교하여 LinkedList의 add() 메소드의 내부 구현은 어떻게 다른지 확인해보자.

## 3. add() - LinkedList

<img width="339" alt="image" src="https://user-images.githubusercontent.com/121920173/230878650-5bdbaa92-6334-41e3-881a-7de0d9674b4b.png">

<img width="170" alt="image" src="https://user-images.githubusercontent.com/121920173/230878455-42428436-16a0-4e4d-a5b4-7558f5f2ad35.png">

LinkedList의 기본 생성자에는 별다른 내용이 없다.  
그리고 기본 생성자에는 `Constructs an empty list`라고 설명이 적혀있다.  

LinkedList에서는 데이터를 추가할 때, add() / addFirst() / addLast() 등의 메소드가 존재하지만 ArrayList와 동일하게 Collection 인터페이스에서 받아 구현한 add() 메소드에 대해서 설명하도록 하겠다.


### 3.1 마지막 인덱스에 값이 추가된 경우

<img width="266" alt="image" src="https://user-images.githubusercontent.com/121920173/230879386-e6fd5a4e-7749-41df-b19c-73d4bd3dd559.png">

<img width="400" alt="image" src="https://user-images.githubusercontent.com/121920173/230879844-1839e109-7d56-49b5-ae97-2c0e9ec0471f.png">

Node 타입의 l을 final로 선언하고, (여기서 l = null이다.)  
여기서 Node는 아래의 사진과 같이 해당 element의 앞, 뒤 데이터에 접근이 가능하도록 구현되어 있다.  
LinkedList의 구조는 흔히 기차처럼 열차의 칸들이 이어져 있다고 말한다.  
열차라고 생각하고 해당 구조를 이해한다면 조금 더 쉽게 이해가 갈 것이다.

<img width="359" alt="image" src="https://user-images.githubusercontent.com/121920173/230880389-52220bd2-f953-4ce6-9ca8-1ccc8fe03862.png">

그리고 newNode라는새로운 객체를 생성하여 prev에는 null, element에는 추가할 데이터, next에는 다시 null값을 넣어준다.  
그리고 last에 newNode를 대입해준다.  
앞에서 확인한 바와 같이 l은 null이므로 first값에 newNode를 대입하게 된다.  
그리고 최종적으로 true를 반환해준다.

위 과정은 LinkedList 0번째 인덱스에 값을 넣어줄 때 발생하는 과정이다.

한번에 와닿지 않는 설명이지만, 위 설명을 열차칸에 비유하여 설명하자면  
해당 값의 앞에 prev라는 칸이 존재하고 뒤에는 next라는 칸이 존재한다고 생각하면 된다.

LinkedList에 add() 메소드의 실행과정은 ArrayList보다 간단해보인다.  
그렇다면 값이 중간에 추가된 경우에는 어떤 일이 일어날까?

### 3.2 값이 중간된 추가된 경우

![image](https://user-images.githubusercontent.com/121920173/231169242-3211f700-f66d-4082-9a88-a36903063607.png)

![image](https://user-images.githubusercontent.com/121920173/231170451-7fa8f248-363e-44d8-8542-8ed8190eacbd.png)

LinkedList의 중간에 값이 추가된 경우를 열차의 칸들로 설명하자면, 추가되는 값을 새로운 칸(노드)으로 생성하고 해당 인덱스 앞 뒤의 값들로 prev, next를 정의해준다고 생각하면 될 것이다.

![linkedList_add](https://user-images.githubusercontent.com/121920173/231172453-6836a2d5-8019-4b7b-8e87-9269fe9863af.png)

## 4. add() 실행 속도 비교

데이터를 추가하는 경우도 2가지로 나누어 비교해 보았다.

1. 순차적으로 데이터를 추가할 경우
2. 중간에 데이터를 추가할 경우

### 4-1. 순차적으로 데이터를 추가할 경우

<img width="584" alt="image" src="https://user-images.githubusercontent.com/121920173/232197523-986dedba-8294-457d-9450-b35e6497f258.png">

`설명하기 앞서 해당 테스트 코드는 정확한 속도 비교를 위해 nanoTime()으로 측정되었다.`

결과를 통해 알 수 있듯이 순차적으로 데이터를 추가할 경우 ArrayList가 LinkedList보다 약 3배 빠르다.

### 4-2. 중간에 데이터를 추가할 경우

그렇다면 중간에 데이터를 추가할 경우 어떤 결과가 나올까?

![image](https://user-images.githubusercontent.com/121920173/232236228-b8bec387-0fb9-410f-82e8-ed297676a02d.png)

![image](https://user-images.githubusercontent.com/121920173/232236573-38655189-5743-4572-8d96-f6959617a059.png)

10000개의 데이터가 있는 list에서는 10개의 데이터를 중간에 추가할 때는 ArrayList와 LinkedList의 속도 차이가 크게 나지 않는다.  
하지만 데이터의 전체 크기 그리고 중간에 삽입되는 데이터가 늘어날수록 ArrayList와 LinkedList의 수행 속도 차이가 확연히 난다는 걸 확인할 수 있다.

순차적으로 추가되는 데이터의 경우 ArrayList가 중간에 데이터가 추가될 경우 LinkedList의 속도가 빠르다는 결론을 얻을 수 있다.

## 5. get()

그렇다면 데이터의 추가 add() 메소드는 여기까지 알아보고 데이터의 삭제 remove() 메소드에 대해 알아보기 이전에 데이터의 조회 get()부터 알아보자.

먼저, ArrayList의 get(index i) 메소드에 대해 알아보자.

<img width="290" alt="image" src="https://user-images.githubusercontent.com/121920173/230922172-153d0382-c738-485b-9339-8821db53c952.png">

<img width="439" alt="image" src="https://user-images.githubusercontent.com/121920173/230922251-6b99fb4c-6097-4b07-8f2e-441c8bb762af.png">

먼저 해당 인덱스의 값이 있는지의 여부를 확인한 다음 해당 데이터의 값을 return 한다.

다음은 LinkedList의 get(index i) 메소드에 대해 알아보자.

<img width="218" alt="image" src="https://user-images.githubusercontent.com/121920173/230922869-41963183-773a-42ac-b611-b73a9708ba01.png">

<img width="366" alt="image" src="https://user-images.githubusercontent.com/121920173/230894537-0f2beb05-7b28-4f45-97bf-ef8b958972df.png">

LinkedList도 동일하게 먼저 해당 인덱스에 값이 있는지를 확인하고, 해당 인덱스의 크기/2보다 작을 경우 if절을 클 경우 else문을 수행하여 해당 인덱스의 값을 리턴한다.

여기서 ArrayList의 LinkedList의 차이는 ArrayList는 해당 인덱스의 element에 바로 접근하는 반면, LinkedList는 해당 리스트의 크기/2 기준으로 해당 인덱스까지 모두 조회한다는 점이다.

get()은 수행 속도 비교를 하지 않아도 ArrayList가 빠르다는 것을 알 수 있다.  
하지만 list의 크기가 작을 수록 둘의 속도 차이는 거의 나지 않을 것이다.

## 6. remove() - ArrayList

1~10 int값을 저장한 list에서 반복문을 수행하는 도중에 3의 배수인 데이터를 삭제하면 예외가 발생할까?

나는 예외가 발생하는지 몰랐다...

그렇다면 실제 테스트 코드를 통해 확인해보자.

<details>
<summary>IndexOutOfBoundsException 이야기</summary>
<div markdown="1">

<img width="706" alt="image" src="https://user-images.githubusercontent.com/121920173/230785738-c36867d7-1e4c-4615-b5d9-9607b6a11562.png">

예외가 발생하였다.  
그것도 `IndexOutOfBoundsException` 예외가..  
`IndexOutOfBoundsException` 예외가 발생했다는 것은 해당 데이터가 삭제되면서 크기 또한 줄어들었다고 예상할 수 있다.

이제 remove() 내부 구조를 확인해보자.

<img width="518" alt="image" src="https://user-images.githubusercontent.com/121920173/230785969-45c44919-160e-42bb-af8c-3f0c1a209fd2.png">

이래서 내부 구조를 한 번은 꼭 뜯어봐야 하나 보다.  
주석에 정답이 바로 적혀있다.  
`Shifts any subsequent elements to the left (subtracts one from their indices).`  
데이터를 삭제하면 삭제된 데이터의 뒷 데이터들이 왼쪽으로 한 칸씩 이동하게 되면서 인덱스 값 또한 -1이 된다는 것이다.

그렇다면 디버깅을 통해 더 상세히 알아보자.

`Objects.checkIndex(index, size)`는 해당 인덱스에 값이 있는지 확인한다. 만약 해당 인덱스에 값이 없다면 `IndexOutOfBoundsException`이 발생한다.

<img width="709" alt="image" src="https://user-images.githubusercontent.com/121920173/230815997-7954371a-ae0e-428e-ac8c-8af066b3a03c.png">

여기서 `oldValue = es[index]`에 삭제될 인덱스의 value를 담아둔 후, 결과값으로 return 한다.

<img width="573" alt="image" src="https://user-images.githubusercontent.com/121920173/230816411-60b548c5-2cdd-42fe-a0df-5faa96a3129e.png">

여기서 if절은 항상 true를 반환할 것이다.  
해당 조건절에 false를 반환하는 경우는 fastRemove()로 넘어가기 전 단계인 `Objects.checkIndex(index, size)` 코드에서 확인이 되기 때문이다.

`System.arraycopy(es, i+1, es, i, newSize-1)`를 통해 배열을 복사한다.

<img width="559" alt="image" src="https://user-images.githubusercontent.com/121920173/230820457-8c868a4c-7b7b-4fa3-97bf-da14bfa1e98a.png">

복사된 es 값을 확인해보면 해당 인덱스의 값이 삭제되고 차례대로 왼쪽으로 index 값이 한 칸씩 이동한 것을 확인할 수 있다.  
원본 배열의 i+1(여기서는 3)부터 newSize-i(여기서는 7)의 갯수만큼 복사된 것을 확인할 수 있다.  
그리고 마지막으로 es배열의 마지막 index값은 index 값이 -1 줄어들어 삭제된 데이터 이므로 null 값을 넣어줬다.

`es[size = newSize] = null`

</div>
</details>

<br>

<img width="620" alt="image" src="https://user-images.githubusercontent.com/121920173/232194579-c82f2cff-0d8c-47e7-bdb5-19ce420d02a1.png">

<img width="503" alt="image" src="https://user-images.githubusercontent.com/121920173/232194504-a993519d-5439-4ff6-8d9c-963021258640.png">

<img width="244" alt="image" src="https://user-images.githubusercontent.com/121920173/232194682-f2523b75-2516-4319-94bf-da44e00e53c3.png">

<img width="385" alt="image" src="https://user-images.githubusercontent.com/121920173/232192557-f640ff7c-34b5-49cc-ae9f-65d9e67ec04f.png">

디버깅을 통해 remove() 내부 코드를 확인해보면 List의 크기에 변화가 생겼으므로 modCount++가 증가하고 해당 인덱스의 값을 삭제하고 iterator의next()를 통해 다음 인덱스 값을 호출할 때, checkForComodification()에서 ConcurrentModificationException 예외가 발생한다.

그렇다면 해당 예외를 발생하지 않게 하는 방법에 어떤 방법이 있을까?  
해당 예외는 iterator의 next()를 호출하면서 발생하는 예외이다.  
가장 쉽게 생각할 수 있는 방법은 일반 for문을 이용하는 방법이다.  
하지만 주석을 통해 설명했듯이 일반 for문에서는 IndexOutOfBoundsException가 발생한다.

IndexOutOfBoundsException와 ConcurrentModificationException가 발생하지 않는 방법에 대해 알아보자.

1. 역순으로 조회
2. removeAll() 사용
3. Iterator의 remove() 사용
4. removeIf() 사용
5. stream.filter() 사용

총 5가지의 방법을 제시하였으나 1번 방법이 어떤 예외를 해결한다라고 보기보다  
5가지 방법 모두 2가지 예외를 발생시키지 않는 방법이다라고 생각하면 좋을 거 같다.

총 5가지 방법을 제시하였으나 결론부터 말하는 추천 사용방법은 4, 5번이다.  
이유는 각 코드를 보면서 이야기해보자.

![image](https://user-images.githubusercontent.com/121920173/232195762-03f52a7f-7745-4196-b1de-90eedc3a91b7.png)

![image](https://user-images.githubusercontent.com/121920173/232196346-826e2574-1a00-4275-8713-df1349ab8d3f.png)

코드의 실행속도와 예외없이 실행되는 것도 중요하지만 놓칠 수 없는 것이 **가독성**이다.  
간단한 코드이기 때문에 1, 2, 3번의 파악이 어려운 것은 아니지만 실제로는 이렇게 간단하게 작성할 수 있는 코드는 몇 없을 것이다.  
1, 2, 3번에 비해 4, 5번에 작성된 코드를 보자.  
깔끔하다!!  
removeIf를 이용한 4번의 코드가 가장 명확하다.  
5번 코드로 작성할 경우 유의할 점은 filter를 통해 나온 결과는 새로운 List로 받아 사용해야 한다는 것이다.  
만약 기존 list의 값이 유지되어야 한다면 5번을 그렇지 않다면 4번으로 사용하면 좋을 거 같다.

이렇게 ArrayList에서 remove() 메소드가 실행될 때 일어나는 내부 코드를 디버깅을 통해 알아보았다.

## 7. remove() - LinkedList

LinkedList에서도 ArrayList와 동일한 테스트를 해보자.

LinkedList에서는 데이터를 삭제할 때, remove() / removeFirst() / removeLast() 등의 메소드가 존재하지만 ArrayList와 동일하게 Collection 인터페이스에서 받아 구현한 remove() 메소드에 대해서 설명하도록 하겠다.

remove() 메소드도 내부에 파라미터로 int형의 index값을 받냐 Object를 받냐에 따라 다르지만, 동일하게 index값을 받아 삭제해주는 로직을 확인해보기로 하겠다.

<img width="545" alt="image" src="https://user-images.githubusercontent.com/121920173/230893549-7d3f5e32-8821-4595-8680-713d0dc32221.png">

주석만으로 확인한 결과 ArrayList와 동일하게 해당 인덱스의 값을 삭제 후, 왼쪽으로 한 칸씩 값이 옮겨 간다는 것을 알 수 있다. 그리고 return 타입 또한 삭제된 데이터를 돌려준다.  
주석만으로는 정확한 내부 구조를 알 수 없으니 다시 한 번 디버깅을 통해 내부 구조를 알아보자.

<img width="506" alt="image" src="https://user-images.githubusercontent.com/121920173/230894151-9211eff4-1f2b-470b-8564-71836d424d20.png">

`checkElementIndex` 메소드는 해당 인덱스의 값의 존재 여부를 확인한다. 만약 없다면 `IndexOutOfBoundsException` 예외를 발생시킨다.

<img width="366" alt="image" src="https://user-images.githubusercontent.com/121920173/230894537-0f2beb05-7b28-4f45-97bf-ef8b958972df.png">

예외없이 돌아올 경우 다음으로 수행되는 `unlink(node(index))` 내부 코드를 보자면,  
먼저 `node(index)`를 보자.

index가 size/2보다 작다면 if절로 아니면 else절이 수행된다.  
first는 0번째 인덱스 값이며, 최종적으로 return 되는 x는 해당 index의 값이 된다.  
else문에서 return 되는 x 또한 해당 인덱스의 값이다.

<img width="418" alt="image" src="https://user-images.githubusercontent.com/121920173/230919526-9b0f2708-305c-4166-bc83-7f01296e60f9.png">

`unlink(Node<E> x)`에서 넘어오는 파라미터의 값은 삭제하려고 하는 인덱스의 참조값이며,  
첫 번째 if절은 해당 인덱스가 첫 번째 값이냐 아니냐의 따른 조건이며, 두 번째 if절은 해당 인덱스가 마지막 값이냐 아니냐의 따른 조건이다.

결론적으로 해당 index의 앞 뒤 값을 재할당 해주고
해당 item값은 null로 할당하고 size는 -1해준다.

## 8. remove() 실행 속도 비교

데이터를 삭제하는 경우도 2가지로 나누어 비교해 보았다.

1. 순차적으로 데이터를 삭제할 경우
2. 중간에 데이터를 삭제할 경우

### 8-1. 순차적으로 데이터를 삭제할 경우

![image](https://user-images.githubusercontent.com/121920173/232237305-415bd14a-8649-40d0-959b-7925467d98ed.png)

50000개 이상의 데이터를 순차적으로 삭제할 경우(해당 테스트에서는 마지막 인덱스부터 순서대로 삭제하였다.) ArrayList와 LinkedList의 수행 속도 차이가 약 2배 정도 난다는 것을 확인할 수 있다.

### 8-2. 중간에 데이터를 삭제할 경우

![image](https://user-images.githubusercontent.com/121920173/232238158-812a0144-6e27-4fb6-afd6-c425bbe066fc.png)

5000개의 데이터가 있는 list에서는 중간의 랜덤 인덱스 값 10개를 삭제할 때는 LinkedList가 ArrayList보다 빠르다는 것을 확인할 수 있다.

순차적으로 삭제되는 데이터의 경우 ArrayList가 중간에 데이터가 삭제될 경우 LinkedList의 속도가 빠르다는 결론을 얻을 수 있다.

## 9. 결론

어떤가?

이때까지 List를 사용할 때는 대부분 ArrayList를 사용해왔다.  
사용하면서 ArrayList의 내부 구조가 어떻게 동작하는지 몰랐는데, 이번 기회를 통해 확실히 완전히 뜯어본거 같다.  
솔직히 생각했던 것보다 훨씬 복잡하고 많은 과정을 거치고 있다는 것을 보고 놀랐다.  
우리는 단순히 데이터를 추가, 삭제만 할 뿐이지 내부 구조가 어떻게 되어 있는지 몰랐기 때문이다.

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