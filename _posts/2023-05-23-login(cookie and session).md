---
title: 로그인 기능 구현(1) - Cookie & Session
date: 2023-05-23
categories: [Cookie, Session]
tags: [Login, Cookie, Session]
---

HTTP는 Stateless(무상태)한 성격을 가지고 있다.
Stateless한 성격 때문에 요청을 할 때마다 응답에 필요한 정보를 매 번 보내줘야 한다.
로그인과 같은 기능 또한 따지고 보면 매 번 요청을 보낸 후 인증을 받아야 하는 것이다.

하지만 우리는 로그인과 같은 기능을 유지하기 위해 Cookie, Session, JWT(Json Web Token)을 이용한 방법을 이용하여 로그인을 유지한다.

이번 포스팅에서는 Cookie와 Session을 이용한 로그인을 구현하고 알아보도록 하자.

## 1. Cookie
![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/74274231-2b14-4837-8182-aa672f390db5)

쿠키를 이용한 방법의 경우, 요청을 보낼 때 데이터를 담아 보내고 응답을 받아 올 때 저장된 쿠키를 응답받아 쿠키 저장소에 저장한다. 이 후, 해당 쿠키의 값이 필요할 때 쿠키 저장소에서 꺼내 사용할 수 있다.

로그인에 필요한 아이디와 비밀번호를 쿠키에 저장한 후, 로그인 인증이 필요한 페이지나 쿠키 데이터가 필요한 경우 쿠키 저장소에서 아이디와 비밀번호를 꺼내 활용할 수 있다.

```java
public void createLoginCookie(Member member, HttpServletResponse response) {  

	Cookie loginIdCookie = new Cookie("loginId", member.getLoginId());  
	Cookie passwordCookie = new Cookie("password", member.getPassword());  

	response.addCookie(loginIdCookie);  
	response.addCookie(passwordCookie);  

}
```
쿠키는 작성된 코드를 보면 알 수 있듯이 간단하게 생성하여 사용할 수 있다.

위 예제 코드와 같이 작성 후, 요청을 보내면 다음과 같이 쿠키가 생성된 것을 확인할 수 있다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/26295596-2978-4498-b4d0-7fc3f6c2fbda)

Expires의 Session은 쿠키의 인증기한을 나타내며, 쿠키는 세션쿠키/영속쿠키가 있다.  
만료 날짜를 생략하면 브라우저 종료시 까지만 유지되는 세션 쿠키가 되고, 만료 날짜를 입력하면 해당 날짜까지 유지되는 영속 쿠키가 된다.

또한 로그아웃은 해당 Cookie의 maxAge 값을 0으로 지정해주면 된다.

```java
public void logoutCookie(HttpServletResponse response) {
 
	Cookie loginIdCookie = new Cookie("loginId", null);  
	Cookie passwordCookie = new Cookie("password", null);  
  
	loginIdCookie.setMaxAge(0);  
	passwordCookie.setMaxAge(0);  
  
	response.addCookie(loginIdCookie);  
	response.addCookie(passwordCookie); 
 
}
```

loginIdCookie와 passwordCookie가 삭제된 것을 확인할 수 있다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/2b34deed-7a94-459c-b044-1235f572c886)

그렇다면 이토록 간단하게 사용자의 ID와 비밀번호를 쿠키에 저장하고 매 요청마다 Cookie 저장소에서 ID와 비밀번호를 확인할 수 있는 간단한 방법이 있는데, 왜 쿠키만을 이용한 로그인 기능은 사용하지 않을까?

저장된 Cookie 정보는 누구나 확인이 가능하고 탈취 할 수 있는 정보이다.  
그렇다보니 위변조가 쉽고 해당 정보를 악용하기 쉬워진다.  
저장된 쿠키를 보면 바로 확인할 수 있듯이 ID와 비밀번호가 바로 노출된다.  
이외에도 다른 개인정보를 입력하여 사용한다면 모두 노출되는 값이라는 것이다.  
특히, 로그인 정보나 개인정보의 경우 노출된다면 어떻게 될까?  
내 메일로 스팸메일이 전송되어 메일 계정이 잠기고, 070 전화를 무척 많이 받게 될 것이다.

이런 Cookie의 단점을 해결하기 위해 우리는 Cookie와 함께 Session을 활용할 수 있다.

## 2. Session

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/3609f351-0c1b-432c-bdb9-dcbf1dda6038)

Session의 경우, 클라이언트에서 보여지는 key값은 임의의 값으로 지정하며 예측 불가능한 값이어야 한다.  
최초 요청 시, 요청을 보낸 정보를 이용하여 서버에서 Session의 value값으로 저장하며 임의의 key값을 돌려보내준다. 그리고 리턴된 sessionId값의 경우 쿠키 저장소에 저장된다.

다시 요청이 들어올 때는 쿠키 저장소의 sessionId를 보내주어 서버에서 해당 sessionId의 value를 조회한다.

Session은 Cookie와 달리 클라이언트에서 보여지는 값으로 로그인 정보, 개인정보를 예측할 수 없어 Cookie의 보안적인 면을 보완해준다.

```java
public static final String SESSION_COOKIE_NAME = "sessionId";  
private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

public void createSession(Member member, HttpServletResponse response) {
  
	String sessionId = UUID.randomUUID().toString();  
	sessionStore.put(sessionId, member);  
  
	Cookie sessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);  
	response.addCookie(sessionCookie);  

}

public void expire(HttpServletRequest request) {  

	Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);  
  
	if (sessionCookie != null) {  
		sessionStore.remove(sessionCookie.getValue());  
	}  

}  
  
private Cookie findCookie(HttpServletRequest request, String cookieName) {
  
	if (request.getCookies() == null) {  
		return null;  
	}  
	
	return Arrays.stream(request.getCookies())  
		.filter(cookie -> cookie.getName().equals(cookieName))  
        .findAny()  
        .orElse(null);  

}
```

login을 통해 session을 생성하면 응답으로 아래와 같이 새롭게 생성된 sessionId를 확인할 수 있다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/6787c597-2e67-4314-8bf0-dc0d7f409981)

`sessionStore.remove();`를 통해 해당 sessionId를 삭제하면 sessionId가 삭제된 것을 확인할 수 있다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/9eeb0bf9-d665-4763-8505-8fa207366b65)

하지만 자바는 좀 더 간편하게  Session을 생성할 수 있는 HttpSession을 제공한다.  
HttpSession을 이용하여 다시 로그인 기능을 구현해보자.

## 3. HttpSession

위에서 우리가 직접 생성한 session과 HttpSession의 생성 방식은 거의 동일하다고 생각하면 된다.  
HttpSession은 sessionId를 자동으로 생성해준다.

위의 코드를 아래와 같이 간단하게 변경할 수 있다.

```java
//session 생성
HttpSession session = request.getSession();  
session.setAttribute(LOGIN_MEMBER, loginMember);

//session 삭제
HttpSession session = request.getSession(false);   
if (session != null) {  
    session.invalidate();  
}
```
`request.getSession(true) or request.getSession()`의 경우 기존 세션이 있을 경우 해당 세션을 반환하고, 없을 경우 새로운 세션을 생성하여 반환한다.  
`request.getSession(false)`의 경우 기존 세션이 있을 경우 해당 세션을 반환하고, 없을 경우 null을 반환한다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/202a3531-33dc-4c04-851e-69c931b87193)

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/b8563ebf-ef79-478b-95d6-57f085c88d98)

HttpSession을 이용하여 위와 같이 JSESSIONID과 생성된 것과 삭제된 것을 확인할 수 있다.

추가적으로 Servlet Session은 Session의 유효시간을 지정하여 유효시간이 지난 Session을 삭제할 수 있다.  
`session.setMaxInactiveInterval(s)`를 이용하여 유효시간을 지정할 수 있으며, 기본값은 1800초(30분)이다. 해당 시간을 너무 과하게 길게 잡을 경우, 사용하는 회원이 증가하였을 때 `Out of Memory`가 발생할 수 있다는 점도 알아두자.  
`session.getLastAccessedTime()`은 최근 Session 접근 시간을 알려주며 해당 시간이 요청 시간과 비교하여 유효 시간을 초과한 경우, 해당 Session을 삭제한다.  
또한 Session도 쿠키에 저장되는 데이터이다. `maxInactiveInterval`을 이용하여 적절한 유효시간을 지정하여 사용하지 않는 Session에 대한 메모리 누수를 방지할 수 있을 것이다.

## 4. 결론

하지만 Session도 결국 저장소에 저장되어야 하는 데이터이다.  
사용하는 사용자의 수가 급증하였을 때의 문제점을 단순하게 해결할 수 있는 방법은 없어 보인다.

그렇다면 해당 문제를 해결할 수 있는 방법은 저장하지 않고 인증을 할 수 있는 방법이 필요해 보인다.  
글 초반에 Cookie, Session과 함께 나열한 JWT(Json Web Token)을 이용해 볼 수 있을 것 같다.  
다음에는 JWT(Json Web Token)가 Cookie, Session과는 어떤 점이 다르고 Cookie, Session의 문제점을 어떻게 해결해 줄 수 있는지 알아보자.

<br>
**출처**  
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-2

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