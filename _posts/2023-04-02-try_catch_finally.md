---
title: finally문에 return 할까? 말까?
date: 2023-04-02
categories: [try_catch_finally]
tags: [try_catch_finally]
---
예외 처리를 해주는 try_catch_finally문에서 각 각 return문을 쓰면 어떤 데이터가 return 될까?

테스트 코드를 통해 확인해보기 전에 내가 생각한 예상 결과는 `어떤 경우에도 finally문은 실행되기 때문에 finally문에 retrun문이 있다면 최종적으로 return되는 값은 finally문 안의 값이 아닐까?`라고 생각했다.

그렇다면 실제 코드를 통해 어떤 결과가 나오는지 알아보도록 하자.

테스트 코드는 아래의 시나리오와 같이 작성하였다.

<img width="500" alt="image" src="https://user-images.githubusercontent.com/121920173/229329855-0463dca6-03a9-49e7-8ef2-2fb1d2d3850e.png">

## 1. finally문 여부에 따른 테스트 코드

<img width="600" alt="image" src="https://user-images.githubusercontent.com/121920173/229298205-43ccb794-5664-4739-a963-ffe9e458b776.png">

<img width="900" alt="image" src="https://user-images.githubusercontent.com/121920173/229298226-7a8f989e-e944-4b58-a101-256c79e905f6.png">

출력된 값을 확인해보면 배열의 크기보다 큰 인덱스에 데이터를 추가하니 예상한 ArrayIndexOutOfBoundsException이 발생하면서 catch문의 return 값을 받아오는 것을 확인할 수 있다.

그렇다면 여기에 finally문이 추가되면 어떤 결과가 나올까?

<img width="600" alt="image" src="https://user-images.githubusercontent.com/121920173/229298453-2a40746b-a84f-4823-88e7-a47ebb8e949a.png">

<img width="900" alt="image" src="https://user-images.githubusercontent.com/121920173/229298481-993555bd-5bdb-4b47-85a0-1f015cac0441.png">

finally문이 추가된 코드에서는 처음 테스트 코드와 같이 ArrayIndexOutOfBoundsException이 발생하지만 return되는 값은 finally문의 데이터라는 것을 확인할 수 있다.

**그런데 이때, catch문에서 예외 처리를 못하는 다른 예외가 발생한다면 어떻게 될까?**

<img width="600" alt="image" src="https://user-images.githubusercontent.com/121920173/229517881-5e99cb1d-bf98-40a8-902a-cf6318cede56.png">

<img width="220" alt="image" src="https://user-images.githubusercontent.com/121920173/229519128-d6ebbf75-d2c1-4369-b274-0cc31dee588e.png">

출력된 결과를 확인해보면 try문에서 발생한 예외가 무시된다.  
해당 코드에서 예상한 결과는 NullPointerException이 발생하는 거였지만,  
finally절에서 return문 때문에 try문에서 발생한 예외는 무시된다.

만약, finally문의 return 구문이 없다면?

<img width="600" alt="image" src="https://user-images.githubusercontent.com/121920173/229520222-e17760cd-0825-4263-8241-59cd1d6afc16.png">

<img width="923" alt="image" src="https://user-images.githubusercontent.com/121920173/229520381-20c8a4f3-ba1b-4b5c-8474-8c1946207078.png">

예상대로 NullPointerException 예외가 발생한 것을 알 수 있다.

## 2. 그렇다면 왜 이런 결과가 나오는 걸까?

finally문은 try_catch문에 예외 발생 여부와 상관없이 **무조건 실행**되어야 하는 구문이다.  
finally문이 필수는 아니지만 finally문을 넣어주게 되면 무조건 실행된다는 것을 잊으면 안된다.

## 3. 그렇다면 finally에는 어떤 코드가 오면 좋을까?

우선 위의 예제로 사용했던 것과 같이 finally문에서 return은 절대 사용하면 안된다.  
**왜일까?**

코드의 결과값으로도 알 수 있듯이 finally문이 포함된 코드의 경우, try_catch문의 return 데이터는 무시되고 finally의 return 값을 가져온다.

아래의 테스트 코드를 다시 보자.

<img width="100%" alt="image" src="https://user-images.githubusercontent.com/121920173/229330338-aaa22732-e9dc-45f5-af66-f400bf344cac.png">

<img width="250" alt="image" src="https://user-images.githubusercontent.com/121920173/229301271-9e4a4fd4-e1dd-4a7e-a14b-22db18d2ee73.png">

**문제는 여기서 발생한다.**  
나는 벚꽃 명소를 추천 받는 데이터를 return 받고 싶었는데, 출력값을 확인해보면 finally문의 값이 retrun된 것을 확인할 수 있다.

**내가 의도한 결과값이 나오지 않는 아주 큰 문제가 발생한 것이다.**   
그러므로 우리는 finally문에서 return을 사용하면 안된다.  
**결과값이 우리가 제어할 수 없게 되어버리기 때문이다.**

그렇다면 어떤 코드가 finally문에 오면 좋을까?  
위에서 설명했듯이 finally문은 예외 발생 여부에 상관없이 실행된다.  
그 말은 try_catch문 실행 이후 공통적으로 수행되어야 하는 코드가 있다면 try_catch문에 각 각 넣는게 아니라 finally문에 추가하여 코드의 중복을 막을 수 있을 것이다.

## 4. 결론

finally문에서는 return문을 사용하지 말자.  
내가 의도하지 않은 결과가 나올 수 있다.  
하지만 try_catch문의 실행 이후 공통적으로 수행되어야 코드가 있다면 finally문을 이용하면 중복 코드가 사라지고 간결하게 나타낼 수 있다.

**참고
https://stackoverflow.com/questions/18205493/can-we-use-return-in-finally-block

<script src="https://utteranc.es/client.js"
        repo="j-jeongeun/github.io.comments"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
