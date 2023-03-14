---
title: String vs StringBuffer/StringBuilder
date: 2023-03-14
categories: [String]
tags: [String]
---
자바의신 String관련 챕터를 읽어보다가 StringBuffer와 StringBuilder 클래스에 대한 이야기가 나왔다.  
실제로 StringBuffer와 StringBuilder를 사용해 본적이 없기 때문에 String 클래스와는 어떤 점이 다르고 어떤 경우에 어떤 클래스를 사용해야 하는지 궁금했다.  
책에서는 두 개의 차이점을 thread-safe냐 unsafe냐 정도로만 간략히 설명해 놓았다. 

**그렇다면, 왜 String이 아니라 StringBuffer/StringBuilder를 사용해야 할까?**  
왜?를 알기 위해 String이 선언되었을 때 일어나는 일을 알아보자.

다음과 같이 새로운 String 객체를 선언하였다.  
String str = "hello";  
'str'이라는 문자열 객체는 heap 영역에 저장된다.  
**String 객체의 특징은 한 번 값이 할당되면 불변성(Immutable)** 이라는 성격을 갖는다.  
예를 들어, 현재 'strs의 주소값이 12345라면,  
다음과 같이 str에 문자열을 더해주면 주소값은 12345가 아닌 새로운 주소값인 56789로 변경된다.  
str = str + " world";  
heap 영역에 새로운 str의 영역이 만들어지면서 기존 str 영역은 GC에 의해 소멸된다.  

이에 반해,  
**StringBuffer/StringBuilder 클래스의 경우, 가변성(mutable) 성격**을 가지고 있다.  
한 번 값이 할당되더라도 다른 값이 할당되면 할당된 공간을 변경한다.  
StringBuffer strBuffer = new StringBuffer("hello");  
strBuffer의 현재 참조 주소값이 12345이라면,  
strBuffer.append(" world");  
위 연산을 수행한 뒤에도 해당 객체의 참조 주소값은 12345로 변하지 않는다.  

<hr>

그렇다면 StringBuffer와 StringBuilder는 어떻게 연산을 처리해줄까?  
StringBuffer와 StringBuilder 클래스를 열어보면 두 클래스 모두 AbstractStringBuilder라는 추상 클래스를 상속받았다.  
문자열을 더해주는 연산을 예로 들자면 StringBuffer와 StringBuilder는 append() 메소드를 이용하여 문자열을 더해준다.  
(삭제할 때는 delete() 메소드를 이용)  

AbstractStringBuilder 클래스의 인스턴스 변수 중,  
value : 문자열의 값을 저장하는 Byte형 배열(byte[] value)  
count : 현재 문자열 크기의 값을 가지는 int 변수(int count)를 이용하여 연산을 처리해준다.

AbstractStringBuilder 클래스의 append 메서드의 구현 코드를 확인해보면,  
현재 저장된 문자열의 길이에서 더해진 문자열의 길이만큼 길이를 늘려준 뒤, 늘린 공간에 추가된 문자열을 넣어준다.  
**-> 그래서 값이 변경되어도 같은 주소 공간을 참조하게 되고, 값이 변경되는 가변성이 된다.**

이를 통해 알 수 있는 점은 String 객체의 경우, 변하지 않는 값을 넣을 때 사용하면 좋다. 연산이 자주 일어나는 문자열의 경우, 연산이 발생할 때마다 새로운 메모리에 저장해주어야 하고, 그 때마다 이전 객체를 GC가 정리해주어야 하기 때문에 성능에 영향을 줄 수 있다.

<hr>

String과 StringBuffer/StringBuilder의 차이에 대해 알아 보았으니,  
StringBuffer와 StringBuilder의 차이를 알아보자.

두 클래스의 가장 큰 차이점은 동기화(Synchronization)의 유무이다.  
StringBuffer는 동기화 키워드를 지원하여 멀티쓰레드 환경에서 안전하다.(thread-safe)  
String 클래스도 불변성으로 멀티쓰레드 환경에서 안전하다.  
StringBuilder는 동기화를 지원하지 않기 때문에 멀티쓰레드 환경에서 사용하는 것은 적합하지 않지만, 동기화를 고려하지 않는 단일 쓰레드 환경에서는 StringBuffer 보다 뛰어나다.  

그렇다면 어떤 경우에 어떤 클래스를 사용하면 좋을까?  
String : 문자열 연산이 적고, 멀티쓰레드 환경일 경우  
StringBuffer : 문자열 연산이 많고, 멀티쓰레드 환경일 경우  
StringBuilder : 문자열 연산이 많고, 단일쓰레드이거나 동기화를 고려하지 않아도 되는 경우  

그렇다면 쓰레드는 어떤 것이고, 쓰레드와 동기화는 어떤 관계가 있는지에 대해 다음 포스팅에서 알아보자.

**참고  
https://dev-jwblog.tistory.com/108

<script src="https://utteranc.es/client.js"
        repo="j-jeongeun/github.io.comments"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
