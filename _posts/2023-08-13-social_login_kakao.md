---
title: 소셜 로그인2 (카카오 편) + 구글
date: 2023-08-13
categories: [로그인]
tags: [카카오 로그인 API]
---

이번에 새롭게 시작한 프로젝트의 로그인 기능은 소셜 로그인 기능(네이버/카카오)을 이용해 보기로 하였다.

* [소셜 로그인 (네이버편)](https://j-jeongeun.github.io/posts/social_login_naver)

## 1. 왜 소셜 로그인을 사용하는가?

별도의 아이디나 비밀번호를 기억할 필요 없이 간편하고 안전하게 해당 서비스에 가입할 수 있어, 가입이 귀찮거나 가입한 계정이 생각나지 않아 서비스를 이탈하는 사용자를 잡을 수 있다.

또한, 개발자에게 사용자의 ID 및 비밀번호를 입력받고 검증하는 과정을 직접 구현하지 않고도 사용자에 대한 인증과 인가를 간편하게 처리할 수 있도록 도와준다.  

##  2. 카카오 API 로그인

### 2.1 카카오 API 로그인이란?

카카오 API를 이용한 로그인은 아래와 같이 진행된다.
![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/132d491a-5ddc-40b4-b4a2-4ab7ccdbbaea)

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/ff93ccd1-8101-46a6-ba22-e5d109629734)

### 2.2 애플리케이션 등록

먼저 사용할 API 등록을 먼저 해주어야 한다.
`내 애플리케이션-애플리케이션 추가하기` 버튼을 눌러 애플리케이션의 기본 정보를 입력한다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/e8c4aac4-1522-4c5a-93b3-8baf25a72eaf)

애플리케이션을 생성한 후, 만들어진 애플리케이션을 누르면 왼쪽 메뉴의 `요약 정보-앱키 or 앱키`에서 확인 할 수 있다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/ffa36689-2ac5-4dfd-8256-545b93de7237)

다음, `플랫폼` 메뉴에서 사용할 플랫폼을 선택하여 도메인을 입력해준 다음, Redirect URI까지 함께 등록해준다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/92b0f74e-6ea3-4334-ae1c-835bb9aabbd2)

RedirectURI는 `제품설정-카카오 로그인`에서 추가 등록할 수 있으며, 해당 메뉴에서 활성화 설정을 ON으로 바꿔주어야 한다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/cc6b9d7a-58ca-44b0-8b3c-f303e5efe998)

다음, `제품 설정-동의항목`에서 받아온 사용자 정보를 설정한다.
카카오는 이름 대신 닉네임 정보를 제공하며, 닉네임, 프로필 사진, 카카오계정 이외의 정보들에 대해 필수 동의를 허용하지 않는다. (개발용 기준)

그래서 우선 내게 필요한 정보들 중, 선택 동의인 성별/연령대에 대해서는 서버에서 다시 한 번 값을 체크하고 넘겨주어야 한다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/ae4327e0-f4df-4ef3-bc80-faf1adced5b7)

여기까지 설정해주면 이제 사용할 준비가 끝났다.

### 2.3 카카오 API 적용하기

앞에서 등록한 애플리케이션을 이용하여 카카오 로그인 API를 적용해보자.

우선, 간단한 로그인 페이지를 만들어보았다.

아래 코드 중, `integrity`는 아래의 링크를 통해서 최신 버전의 스크립트를 복사하면 자동으로 생성된다.  
[카카오 최신버전 js 다운로드](https://developers.kakao.com/docs/latest/ko/sdk-download/js)

```javascript
<script src="https://t1.kakaocdn.net/kakao_js_sdk/2.3.0/kakao.min.js" integrity="sha384-70k0rrouSYPWJt7q9rSTKpiTfX6USlMYjZUtr1Du+9o4cGvhPAWxngdtVZDdErlh" crossorigin="anonymous"></script>
...
<body>
	<a id="kakao-login-btn" href="javascript:loginWithKakao()">  
		<img src="https://k.kakaocdn.net/14/dn/btroDszwNrM/I6efHub1SN5KCJqLm1Ovx1/o.jpg" class="login_icon" width="323"/>  
	</a>
</body>
...
<script>
Kakao.init(kakaoScriptKey);  
function loginWithKakao() {  
	Kakao.Auth.authorize({  
		redirectUri: callbackUrl,  
		state: 'userme',  
	});  
}
</script>
```

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/98f32063-7e81-4b20-9a1a-e4f0999166dd){: width="500"}

로그인 버튼을 누르면 아래와 같이 애플리케이션에 등록했던 내용들을 확인 할 수 있다.

이 때, 만약 아래와 같은 오류가 발생한다면 아래의 링크를 통해 에러 코드별로 원인을 파악할 수 있다.  
[카카오 에러 코드 상세 확인](https://developers.kakao.com/docs/latest/ko/kakaologin/trouble-shooting#code)

`KOE004` 에러 코드의 경우, 로그인 비활성화 상태에서 로그인을 요청하면 발생하는 에러코드이다.

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/5c05d957-58f2-4205-aae8-161df9d02153){: width="500"}

다시 애플리케이션 설정에서 해당 애플리케이션을 활성화 해주면 정상적으로 회원가입 진행 화면이 뜨는 것을 확인할 수 있다. 

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/fc375aaf-7eeb-41ef-93a7-5e35b7fc1399)

```javascript
<script src="https://t1.kakaocdn.net/kakao_js_sdk/2.3.0/kakao.min.js" integrity="sha384-70k0rrouSYPWJt7q9rSTKpiTfX6USlMYjZUtr1Du+9o4cGvhPAWxngdtVZDdErlh" crossorigin="anonymous"></script>
...
<body>
	<a id="kakao-login-btn" href="javascript:loginWithKakao()">  
		<img src="https://k.kakaocdn.net/14/dn/btroDszwNrM/I6efHub1SN5KCJqLm1Ovx1/o.jpg" class="login_icon" width="323"/>  
	</a>
</body>
...
<script>
Kakao.init(kakaoScriptKey);  
function loginWithKakao() {  
	Kakao.Auth.authorize({  
		redirectUri: callbackUrl,  
		state: 'userme',  
	});  
}
</script>
```

(실제 요청 URL : `https://kauth.kakao.com/oauth/authorize`)
````
ex)
https://kauth.kakao.com/oauth/authorize?response_type=code&client_id=${REST_API_KEY}&redirect_uri=${REDIRECT_URI}
````

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/f112a11e-7d0c-42ae-af9e-67c0159f1c3c)

로그인에 성공한다면, 애플리케이션에 설정해둔 Callback URL로 간다.  

이 때, 성공 여부에 따라
-   API 요청 성공시 : `http://콜백URL?code=${AUTHORIZE_CODE}`
-   API 요청 실패시 : `http://콜백URL?error=access_denied&error_description=User%20denied%20access`  

로 반환되며, error 파라미터를 가지고 있다면 로그인 페이지로 그렇지 않다면 성공 화면 페이지로 전환되도록 하였다.

Callback 페이지에서 return된 code 파라미터 값을 이용하여 token을 받고, 사용자 정보까지 요청하면 로그인한 사용자의 정보를 확인할 수 있다.
````
ex)
1) token 발급
https://kauth.kakao.com/oauth/token?grant_type=authorization_code&client_id=${client_id}&redirect_uri=${redirect_uri}&code=${code}

2) 사용자 정보 가져오기
https://kapi.kakao.com/v2/user/me
(헤더에 1)에서 발급받은 token을 보내야함)
````

참고 코드는 해당 링크를 확인하자.  
[카카오 로그인 API 샘플코드](https://developers.kakao.com/tool/demo/login/userme)

##  3. 구글 API 로그인

안타깝게도 현재 구글은 자바스크립트를 이용한 로그인 API 서비스를 중단하였다. 😢

이번 프로젝트에서 로그인을 간편하게 처리할 수 있는 장점때문에 소셜 로그인 API를 사용할 목적이었기 때문에 이번에는 구글 로그인을 제외하기로 하였다. 

![image](https://github.com/j-jeongeun/j-jeongeun.github.io/assets/121920173/664cbcfe-bc71-4d9a-95ab-e6eac5f57b85)

##  4. 마무리

소셜 로그인 API는 이번에 처음 사용해보았는데, 정말 간단한 방법을 통해 회원가입/사용자 정보 조회까지 되는 것을 확인해보았다.

앞에서 작성한 네이버 로그인 API를 이용한 방법보다 카카오 로그인 API가 조금 더 복잡해 보이는 거 같긴 해도 실제로 구현하고 사용되는 건 거의 비슷하다고 생각이 들었다.

개인 프로젝트에서 사용되다 보니 제약 사항이 조금씩은 있지만 개발용도로 사용하기에 아직까지는 문제가 없어 보인다.

**참고**  
[카카오 로그인 개발가이드](https://developers.kakao.com/docs/latest/ko/kakaologin/common)

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