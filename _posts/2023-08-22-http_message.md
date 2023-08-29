---
title: http 라이브러리
date: 2023-08-22
categories: [HTTP 라이브러리]
tags: [HTTP 라이브러리]
---

개인 프로젝트를 진행하면서 외부 데이터 API를 호출해야 하는 경우가 생겼다.  
http 통신을 주고 받는 게 기본적으로 알고 있어야 하는 사항일 수 있지만, 기존 회사의 업무를 진행하면서 외부 API를 사용하는 경우가 거의 없어서 통신을 주고 받는 어떤 방법들이 있고 어떤 방법을 써야 하는지 몰랐다.

그래서 이번에 http 통신을 하게 해주는 여러 라이브러리들을 알아보고 프로젝트에 적용해보기로 하였다.

## 1. HTTP 라이브러리 소개

우선, 가장 오래 사용되어 온 `HttpURLConnection`부터 가장 직관적이고 간편한 `OpenFeign`까지 5가지에 대해서 설명하겠다.

-   HttpURLConnection
-   HttpClient
-   RestTemplate
-   WebClient
-   OpenFeign

각 방법에 대해 설명하기에 앞서 모든 요청 URL은 FAKE DATA를 JSON형식으로 반환해주는 `https://jsonplaceholder.typicode.com`을 이용하여 테스트 코드를 작성하였다.  
또한, 테스트 코드는 `GET` 방식의 요청만 작성하였으며, 헤더나 다른 옵션들을 설정하는 것을 최소화하였다.

## 2. HttpURLConnection

`HttpURLConnection`은 자바에서 제공하는 가장 기본적인 방법이면서 JDK1.1 부터 제공된 방법이다.

설명보다 우선 아래의 간단한 테스트 코드를 보자.

```java
@Test  
void urlConnectionTest() throws IOException {  
	URL url = new URL(TEST_URL);  
  
	HttpURLConnection con = (HttpURLConnection) url.openConnection();  
	con.setRequestProperty("Content-Type", "application/json");  
	con.setRequestMethod(HttpMethod.GET.name());  
	con.setDoOutput(true);  
  
	BufferedReader br = new BufferedReader(new InputStreamReader(con.getInputStream()));  
  
	String inputLine;  
	StringBuffer sb = new StringBuffer();  
	while ((inputLine = br.readLine()) != null) {  
		sb.append(inputLine);  
	}  
    br.close();
    con.disconnect(); // con.close();
}
```

간단한 요청 코드이지만 Connection을 맺고 응답값을 처리해주는 부분의 코드가 길어지면서 가독성이 떨어진다. 그리고 맺은 Connection을 종료하거나 재사용하기 위해 `close()/disconnect()` 처리가 필요하다.
또, 응답값을 내가 원하는 타입으로 받기 위해 한 번 더 후처리가 필요하다.

매 요청마다 Connection을 맺고 종료하고 반환값을 처리하기 위해 해당 코드들을 반복하다 보면 자연스레 코드는 끝도 없이 늘어나고 중복되는 코드는 증가할 것이다.  
이런 `HttpURLConnection`의 단점을 보완하여 나온 방법이 `HttpClient`이다.

## 3. HttpClient

여기서 사용할 HttpClient는 Apache에서 제공하던 HttpClient가 아닌 JDK11에서 추가된 방법이다.  
(Apache에서 제공하던 HttpClient는 HttpComponents로 변경되어 관리되고 있다.)

```java
@Test  
void httpClientTest() throws IOException, InterruptedException {  
	HttpClient client = HttpClient.newBuilder()
		.version(Version.HTTP_1_1)
		.build();
	
	HttpRequest request = HttpRequest.newBuilder()  
		.uri(URI.create(TEST_URL))  
        .GET()  
        .build();  
  
  HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
}
```

`HttpURLConnection`과 동일한 요청의 테스트 코드이지만, 코드의 양이나 가독성이 확실히 좋아진 것을 확인할 수 있다.

또한, `HttpClient`는 HTTP 1/2를 모두 지원하며, 동기/비동기 방식 모두를 지원한다.

아래 예제 코드와 같이 동기/비동기 요청에 따른 차이점을 확인할 수 있다.

```java
// HttpClient.java 클래스 내 샘플 코드 //

// Synchronous Example
HttpClient client = HttpClient.newBuilder()
	.version(Version.HTTP_1_1)
	.followRedirects(Redirect.NORMAL)
	.connectTimeout(Duration.ofSeconds(20))
	.proxy(ProxySelector.of(new InetSocketAddress("proxy.example.com", 80)))      		
	.authenticator(Authenticator.getDefault())
	.build();

HttpResponse<String> response = client.send(request, BodyHandlers.ofString()); 
System.out.println(response.statusCode());
System.out.println(response.body());  

// Asynchronous Example
HttpRequest request = HttpRequest.newBuilder()
	.uri(URI.create("https://foo.com/"))
	.timeout(Duration.ofMinutes(2))
	.header("Content-Type", "application/json")       	
	.POST(BodyPublishers.ofFile(Paths.get("file.json")))
	.build();

client.sendAsync(request, BodyHandlers.ofString())
	.thenApply(HttpResponse::body)
	.thenAccept(System.out::println);
```

## 4. RestTemplate

스프링에서 제공하는 방법이며, 단어 그대로 RESTful한 코드를 짤 수 있는 템플릿을 지원한다.  
HttpClient API 보다 직관적이고 사용하기 편하다는 장점이 있지만, 비동기는 지원하지 않는다.

`RestTemplate` 클래스에 포함된 내용을 살펴보며, 보다 나은 코드와 비동기 요청을 원하면 `RestTemplate` 보다 `WebClient`을 사용하는 것을 추천한다고 적혀있다. 그렇다고 무조건 `WebClient`을 사용하는 것이 아닌 적용하고자 하는 프로젝트의 규모와 사용용도에 따라 적합한 방법을 선택해야 한다.   

>NOTE: As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the org.springframework.web.reactive.client.WebClient which has a more modern API and supports sync, async, and streaming scenarios.  
>참고: 5.0부터 이 클래스는 유지 관리 모드에 있으며 앞으로는 사소한 변경 요청과 버그만 허용됩니다. 보다 현대적인 API를 갖추고 동기화, 비동기 및 스트리밍 시나리오를 지원하는 org.springframework.web.reactive.client.WebClient 사용을 고려해 보세요.

```java
@Test  
void restTemplateTest() {  
    RestTemplate restTemplate = new RestTemplate();  
	HttpHeaders headers = new HttpHeaders();
	headers.set("accept", MediaType.APPLICATION_JSON_VALUE);
	HttpEntity<?> entity = new HttpEntity<>(headers);
  
	ResponseEntity<String> response = restTemplate.exchange(URI.create(TEST_URL),
		HttpMethod.GET, entity, String.class);
}
```

## 5. WebClient

>Spring WebFlux includes a client to perform HTTP requests with. `WebClient` has a functional, fluent API based on Reactor, see [Reactive Libraries](https://docs.spring.io/spring-framework/reference/web-reactive.html#webflux-reactive-libraries), which enables declarative composition of asynchronous logic without the need to deal with threads or concurrency.  
>Spring WebFlux에는 HTTP 요청을 수행하는 클라이언트가 포함되어 있습니다. `WebClient`Reactor를 기반으로 하는 기능적이고 유창한 API가 있습니다. [Reactive Libraries를](https://docs.spring.io/spring-framework/reference/web-reactive.html#webflux-reactive-libraries) 참조하세요 . 이는 스레드나 동시성을 처리할 필요 없이 비동기 논리의 선언적 구성을 가능하게 합니다.

`WebClient` docs에서 확인할 수 있듯이 `WebClient`은 스프링 `WebFlux`에서 제공되는 기능이다. 그 말은 `WebFlux`가 뭔지 알아야 하고, `WebFlux`를 모른다면 다른 방법들에 비해 러닝커브가 더 오래 걸릴수도 있다는 말이다.

`RestTemplate`와 `WebClient`와 비교 대상으로 자주 거론되는데, 둘의 차이를 간단히 비교하자면 아래의 표와 같이 설명할 수 있겠다.

|RestTemplate|WebClient|
|---|---|---|
|멀티 Thread|싱글 Thread|
|Blocking|Non-Blocking|
|동기|동기/비동기|

```java
@Test  
void webClientTest() {  
    String result = WebClient.builder()  
        .baseUrl(TEST_URL)  
        .build().get()  
        .uri(URI.create(TEST_URL))  
        .retrieve()  
        .bodyToMono(String.class)  
        .block();  
}
```

## 6. OpenFeign

`OpenFeign`은 `Netflix`에서 만든 `선언적`인 HTTP Client 라이브러리이다.  
`Netflix`에서 개발하고 내부적으로 사용하고 있었으나, 이후 사용 및 개발을 중단하고 오픈소스로 바뀌었다.

`OpenFeign`은 `인터페이스`를 생성하여 `annotation`을 선언하여 사용한다. 위에서 작성한 다른 예제 코드와 달리 아래의 예제 코드는 확실히 간결해졌다.

하지만 `인터페이스`와 `annotation`만을 이용하여 http 통신을 주고 받다보니 `OpenFeign`을 이용한 테스트 코드를 짜기에는 어려움이 있다는 단점이 있다.

```java
@FeignClient(name = "test-api", url = "https://jsonplaceholder.typicode.com")  
public interface TestService {  
  
    @GetMapping("/posts")  
    String call();  
 
}
```

## 7. 결론

가장 많이 쓰이는 http 라이브러리 5가지에 대해서 알아 보았다.  
이 외에도 `OkHttp`나 `Retrofit` 등과 같이 http 통신을 위한 여러 라이브러리가 있다.

위 방법들 중, 어떤 방법이 정답이고 무조건 이 방법을 써야 한다는 건 없지만,
사용하고자 하는 프로젝트와 용도, 사용자 수 등과 같은 조건 등을 따져 적합한 방법을 찾아 사용하면 좋을 것 같다.

-   HttpURLConnection
-   HttpClient
-   RestTemplate
-   WebClient
-   OpenFeign

예를 들어, 소개한 5가지 방법 중에서 `HttpURLConnection`와 `HttpClient`는 잦은 외부 API 호출 시, 코드의 가독성을 떨어뜨릴 수 있기 때문에, 이 2가지 방법보다는 `RestTemplate` 방법을 추천한다.  
하지만, `RestTemplate`는 동기방식만 지원하기 때문에 비동기 통신이 필요하다면 `WebClient`을 이용한 방법을 추천하며,
`나는 코드가 지저분해지는게 싫어! 간편하게 쓰고 싶어!!`라고 한다면 `OpenFeign` 방법을 추천한다.  
하지만, 라이브러리 소개글에서도 말했듯이 `OpenFeign`은 테스트 코드를 짜기에는 어렵다는 단점이 있다.  
또한, 나처럼 HTTP 라이브러리를 처음 써본다면 `OpenFeign`이 간편하고 깔끔해보이지만, 다른 라이브러리들에 비해 HTTP 통신의 요청/응답이 코드에서 보이지 않아 너무 생략화된 방법이라고 생각이 들 수도 있을거 같다.

**출처**  
https://jie0025.tistory.com/531  
https://docs.spring.io/spring-framework/reference/web/webflux-webclient.html

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