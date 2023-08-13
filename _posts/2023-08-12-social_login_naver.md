---
title: 소셜 로그인1 (네이버 편)
date: 2023-08-12
categories: [로그인]
tags: [네이버 로그인 API]
---

이번에 새롭게 시작한 프로젝트의 로그인 기능은 소셜 로그인 기능(네이버/카카오)을 이용해 보기로 하였다.

* [소셜 로그인 (카카오편)](https://j-jeongeun.github.io/posts/social_login_kakao)

## 1. 왜 소셜 로그인을 사용하는가?

별도의 아이디나 비밀번호를 기억할 필요 없이 간편하고 안전하게 해당 서비스에 가입할 수 있어, 가입이 귀찮거나 가입한 계정이 생각나지 않아 서비스를 이탈하는 사용자를 잡을 수 있다.

또한, 개발자에게 사용자의 ID 및 비밀번호를 입력받고 검증하는 과정을 직접 구현하지 않고도 사용자에 대한 인증과 인가를 간편하게 처리할 수 있도록 도와준다.  

##  2. 네이버 API 로그인

### 2.1 네이버 API 로그인이란? 

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/99fce947-939b-4178-b413-bfe5b256d139)

### 2.2 애플리케이션 등록

먼저 사용할 API 등록을 먼저 해주어야 한다.
`Application-애플리케이션 등록` 메뉴에서 본인이 사용할 정보들을 입력한다. 

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/b9bd1324-282a-4d9e-885f-44631c8f38fb)
![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/f1090c1e-8971-4d60-847d-e466b4096cfb)

서비스명과 해당 서비스에서 필요로 하는 정보들을 체크해준다.  
(사용 API를 체크할 때는 무분별하게 모든 정보를 선택하는 것보다 필요한 정보들만 체크해주는게 좋다. 무분별한 데이터 수집은 회원 이탈을 야기할수도 있다.)  
그리고 해당 서비스의 URL과 로그인 성공 시 반환할 Callback URL까지 입력해주면 된다.

등록완료 후, 내 애플리케이션에서 해당서비스를 확인하면 발급된 ClientID와 Client Secret을 확인할 수 있다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/957dc56f-6aca-4c93-9998-b7a7d8fa2c66)

### 2.3 네이버 API 적용하기

앞에서 등록한 애플리케이션을 이용하여 네이버 로그인 API를 적용해보자.

우선, 간단한 로그인 페이지를 만들어보았다.

```javascript
<script type="text/javascript" src="https://static.nid.naver.com/js/naverLogin_implicit-1.0.3.js" charset="utf-8"></script>
...
<body>
	<div id="naver_id_login"></div>
</body>
...
<script>
var naver_id_login = new naver_id_login(naverClientId, callbackUrl);  
  
var state = naver_id_login.getUniqState();  
naver_id_login.setButton("green", 3, 70);  
naver_id_login.setState(state);  
naver_id_login.init_naver_id_login();
</script>
```

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/d3c59bd9-eb1e-4f6a-b7ad-44469f790088){: width="500"} | ![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/00dd15eb-fece-4c35-83a1-bc27603637a7){: width="500"}

로그인 버튼을 클릭하면 우리가 많이 봐왔던 네이버를 이용한 로그인 동의 화면을 만날 수 있다.  

(실제 요청 URL : `https://nid.naver.com/oauth2.0/authorize`)
````
ex)
https://nid.naver.com/oauth2.0/authorize?response_type=code&client_id=CLIENT_ID&state=STATE_STRING&redirect_uri=CALLBACK_UR
````

애플리케이션 설정 시, 나는 `이름/성별/연령대`만 필수로 선택하였기 때문에 해당 정보들만 필수항목으로 설정된 것을 확인할 수 있다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/f112a11e-7d0c-42ae-af9e-67c0159f1c3c)

로그인에 성공한다면, 애플리케이션에 설정해둔 Callback URL로 간다.  

이 때, 성공 여부에 따라
-   API 요청 성공시 : `http://콜백URL/redirect?code={code값}&state={state값}`
-   API 요청 실패시 : `http://콜백URL/redirect?state={state값}&error={에러코드값}&error_description={에러메시지}`  

로 반환되며, error 파라미터를 가지고 있다면 로그인 페이지로 그렇지 않다면 성공 화면 페이지로 전환되도록 하였다.

**이 때, Callback 페이지에서 `접근 토큰 발급 요청 / 접근 토큰을 이용하여 프로필 API 호출`을 해야 사용자의 정보를 조회할 수 있으나 아래 예제 코드와 같이 바로 사용자 프로필 조회를 하여 `naver_id_login.getProfileData('name');` 와 같이 조회하여도 사회자 정보를 조회할 수 있다.**

```javascript
<script type="text/javascript" src="https://static.nid.naver.com/js/naverLogin_implicit-1.0.3.js" charset="utf-8"></script>
...
<body>
	<h1>login success!!</h1>
	<p>name : <input type="text" id="name"></p>  
	<p>gender : <input type="text" id="gender"></p>  
	<p>age : <input type="text" id="age"></p>
</body>
<script>
...
// 네이버 사용자 프로필 조회  
naver_id_login.get_naver_userprofile("naverSignInCallback()");  
  
// // 네이버 사용자 프로필 조회 이후 프로필 정보를 처리할 callback function
function naverSignInCallback() {  
	const id = naver_id_login.getProfileData('id');
	const name = naver_id_login.getProfileData('name');
	const gender = naver_id_login.getProfileData('gender');
	const age = naver_id_login.getProfileData('age');
}
...
</script>
```

참고 코드는 해당 링크를 확인하자.  
[네이버 로그인 API 샘플코드](https://developers.naver.com/docs/login/web/web.md)


##  3. 마무리

소셜 로그인 API는 이번에 처음 사용해보았는데, 정말 간단한 방법을 통해 회원가입/사용자 정보 조회까지 되는 것을 확인해보았다.

**참고**  
[네이버 로그인 개발가이드](https://developers.naver.com/docs/login/devguide/devguide.md#%EB%84%A4%EC%9D%B4%EB%B2%84%20%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B0%9C%EB%B0%9C%EA%B0%80%EC%9D%B4%EB%93%9C)

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