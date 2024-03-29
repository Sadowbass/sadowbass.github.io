---
title: RestTemplate의 TPS 저하 이슈

categories:

- Problem Solving

tags:

- HTTP
- HttpClient
- RestTemplate
- Http Connection

last_modified_at: 2023-04-02T11:22:58+09:00

---

예전 모 은행의 프로젝트 진행 중 아웃바운드 통신을 위한  
API Gateway(Forward Proxy와 흡사한)를 혼자서 만들었던 적이 있습니다.

사용 중이던 프레임워크에서 특정 엔드포인트에 연결하려면 url을 미리 등록을 해야 하는데  
REST API에서 id나 기타 여러 가지 동적으로 변하는 수십만 고객의 url을 어떻게 미리 등록하겠습니까...  
그래서 프로젝트 진행 중에 저희가 게이트웨이를 하나 만들어서 그쪽으로 요청하면 게이트웨이에서 원하는 url로 요청하는 앱이었습니다.  
`내부 시스템 -> 외부 기관` 에서 `내부 시스템 -> 게이트웨이 -> 외부 기관`이 된 것이지요

자바 11과 스프링 부트를 이용한 그리 복잡하지 않은 앱이었는데 잘 작동하고 로그도 잘 찍히고 통신도 멀쩡하고  
아주 잘 동작하는 애플리케이션으로 보였습니다.

*문제는 통합테스트 때 발생했으니...*  
개발단계에서의 테스트에서 끽해야 수십 개의 요청만 처리하다가 동시에 수천 건의 요청을 날리면 어느 순간 엄청난 양의 타임아웃이 발생한 것이었습니다.  
처음엔 언더토우의 코어, 쓰레드를 만지면서 지켜봤는데 도저히 해결되지 않았습니다.

이때부터 수많은 문서들과 stackoverflow를 뒤져보기 시작했습니다.  
was의 쓰레드가 문제다, 서버 대역폭, 성능의 문제다 등 여러 가지 문제점들을 보고있지만 서버 성능을 내가 뭘 어떻게 할 수 있는 것도 아니고  
아무리 개발 서버라지만 그 큰 은행 서버가 이 정도를 못 버티는 게 말이 안 된다고 생각되던 그때!

HttpConnectionPool에 대한 얘기를 처음 듣게 됩니다.  
http는 커넥션을 매번 맺고 끊는 것이 아니었나?  
커넥션 풀이 무슨 소용이지? 하고 글을 유심히 보기 시작했습니다.  
[[docs]](https://hc.apache.org/httpcomponents-client-4.5.x/current/httpclient/apidocs/org/apache/http/impl/conn/PoolingHttpClientConnectionManager.html)
[[(mdn)connection management]](https://developer.mozilla.org/ko/docs/Web/HTTP/Connection_management_in_HTTP_1.x)

결과적으로 http/1.0에서 지속 연결을 위한 keep-alive connection이 생겼고 http/1.1에서는 keep-alive connection이 기본값이 되었다는 내용이었습니다.  
생각해보니 당연한 일이었습니다. TLS(SSL) 핸드셰이크는 최초에 한 번만 이루어지고 이후 연결에서는 최초에 이루어진 내용으로 통신해서 사용한다고 진작 공부를 했었는데  
왜 http는 매번 연결을 끊는다고 생각한 것인지...

이론적인 부분은 그렇다 치고 일단 당장 급한 해결책이 필요했습니다.  
확인해보니 `RestTemplate`을 생성할 때 파라미터로 `ClientHttpRequestFactory`를 넣어줄 수 있습니다.  
그리고 그 `ClientHttpRequestFactory`를 생성할 때는 `Apache HttpClient`를 넣어줄 수 있는데  
아파치 클라이언트를 생성할 때 `ConnectionManager`를 만들어서 생성할 수 있습니다.  
이때 `ConnectionManager`는 가장 기본적인 매번 커넥션을 맺고 끊는 `BasicHttpClientConnectionManager`과 커넥션풀을
이용하는 `PoolingHttpClientConnectionManager` 두 가지로 나뉘는데  
이 `PoolingHttpClientConnectionManager`를 사용하면 간단히 해결되는 문제였습니다. (내부구조는 전혀 간단하지 않습니다...)

`ConnectionManager`를 만들 때 또 `SSlContext`나 이것저것 들어가지만 여기서는 생략하겠습니다.

`ClientHttpRequestFactory`없이 기본 생성자로 RestTemplate를 생성하면 통신을 위해 이용하는 클래스는 자바의 `HttpURLConnection`를 사용하게 됩니다.  
구현체는 sun에서 구현한 구현체입니다.  
기본 RestTemplate도 keep-alive로 열린 소켓이 있으면 캐시 해서 사용하나 커넥션풀과는 다르고, 계속해서 포트를 열고, 닫고 연결을 맺다 보니 time_wait되는 포트가 늘어갈 경우 지속적인 연결 에러가
발생합니다.

![keep-alive 만료전의 커넥션이 있으면 있으면 캐시해놓은 커넥션을 이용하기는 한다](/assets/images/problem/poolingconnectionmanager/1.png)
*[기본 생성자로 생성한 RestTemplate로 요청했을때의 로그]*  
*[keep-alive 만료전의 커넥션이 있으면 있으면 캐시해놓은 커넥션을 이용하기는 한다]*

본격적으로 `PoolingHttpClientConnectionManager`를 이용하는 RestTemplate를 구현해보겠습니다.

```java
    @Bean
public RestTemplate poolingRestTemplate(){
  // PoolingHttpClientConnectionManager를 생성, 이 예시에서는 Registry 같은 풀링과 관계없는 설정은 하지 않는다.
  PoolingHttpClientConnectionManager poolingHttpClientConnectionManager=new PoolingHttpClientConnectionManager();
  poolingHttpClientConnectionManager.setDefaultMaxPerRoute(20);
  poolingHttpClientConnectionManager.setMaxTotal(100);

  // 위에서 만든 커넥션 매니저로 HttpClient를 생성한다.
  CloseableHttpClient httpClient=HttpClients.custom().setConnectionManager(poolingHttpClientConnectionManager).build();

  // HttpClient를 이용해서 HttpComponentsClientHttpRequestFactory를 만들어서 RestTemplate를 생성한다.
  HttpComponentsClientHttpRequestFactory httpComponentsClientHttpRequestFactory=new HttpComponentsClientHttpRequestFactory(httpClient);

  return new RestTemplate(httpComponentsClientHttpRequestFactory);
  }
```

RestTemplate는 Pooling을 사용하던 Basic을 사용하던 ThreadSafe 하다고 하니 DI 받아서 사용하도록 Bean으로 등록해서 사용합니다.  

![pooling 최초요청로그](/assets/images/problem/poolingconnectionmanager/2.png)
*[최초 요청시 풀에 어떠한 커넥션도 존재하지 않아 route allocate에 새로운 커넥션을 맺는다]*
![pooling 최초응답로그](/assets/images/problem/poolingconnectionmanager/3.png)
*[response의 keep-alive 헤더에 지정된 시간큼 유지된다는 로그. 일정시간마다 커넥션을 체크하게 할수있고. 요청시 커넥션이 살아있는지 확인하는 두가지 방법이 있다]*
![pooling 두번째요청로그](/assets/images/problem/poolingconnectionmanager/4.png)
*[두번째 요청시 total available로 사용가능한 커넥션이 보이고 그것을 사용해서 요청하는 것을 확인할 수 있다]*
![pooling 두번째응답로그](/assets/images/problem/poolingconnectionmanager/5.png)
*[최초 response와 같이 연결을 유지한 커넥션을 풀에 반납한다]*

Wire Shark로 실제 커넥션이 어떻게 이루어지는지도 확인해 보겠습니다.  
응답을 10초 뒤에 주도록 한 뒤 5번의 요청을 동시에 날린 케이스입니다.
![최초 요청시](/assets/images/problem/poolingconnectionmanager/wire1.png)
*[5번의 새로운 tcp 커넥션을 맺는다]*

![두번째 요청시](/assets/images/problem/poolingconnectionmanager/wire2.png)
*[똑같이 5번의 요청을 하지만 새로운 커넥션을 맺지 않고 맺어져있는 소켓을 이용한다]*

확실히 성능의 개선이 있었는지 nGrinder를 통한 부하 테스트도 진행하였습니다.  
당시의 이미지는 이미 소실 되어 새로 테스트를 진행하였으나 부하를 발생시키는 곳과 부하를 받는 곳이 조그마한 개인 pc이며 전부 같은 서버이기에 정확한 테스트는 아닙니다.  
실제로 제대로 된 서버에 부하를 넣었을 때도 비슷한 결과가 나온다는 정도로만 봐주시면 감사하겠습니다.  
테스트 환경은 vuser 1000개 (10 process, 100 thread)로 진행하였습니다. (제 노트북이 이걸 못 버티고 버벅대서 아쉽습니다)

![일반 RestTemplate](/assets/images/problem/poolingconnectionmanager/default.png)
*[일반 RestTemplate]*  
*[실제 업무에서는 http가 아닌 https였으며 mTls까지 적용되어 TPS가 20대까지 떨어졌었다]*  
*[TPS는 테스트 환경을 고려하면 이해할 수 있다. 다만 저 에러의 숫자는 도저히 용납할 수 없다!]*

![Pooling RestTemplate](/assets/images/problem/poolingconnectionmanager/pooling.png)
*[Pooling을 이용한 RestTemplate]*  
*[TPS도 올랐지만 역시 안정적인 커넥션 풀로 에러가 확연히 줄어들었다]*  
*[단순 http도 이 정도의 성능 차이가 벌어지는데 Tls 그것도 양방향 인증하는 mTls였던 실제 서버에서는 더욱 엄청난 차이가 벌어졌다]*

이렇게 수정하고 다시 은행의 개발 서버에 올려서 테스트를 진행하였고  
다행히 아무런 문제 없이 통과 후 운영 서버까지 배포가 되었습니다.

저 때는 제가 1년 차도 되지 않았던 시절이었고 스프링도 http도 정확히 아는 게 아니었던 만큼 알고 있던 RestTemplate을 사용했는데  
지금이라면 다른 HttpClient 종류를 찾아서 쓸 거 같습니다.  
비동기도 되고 커넥션 풀도 관리해주는 좋은 클라이언트가 많더라고요.
