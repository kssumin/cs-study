## CORS()
"다른 출처(cross - origin)"에 resource를 요청할 때 지켜야하는 정책

### 출처(origin)
* url에서 protocol, host, port까지를 의미한다.
<img width="749" alt="image" src="https://github.com/kssumin/cs-study/assets/88534959/731aea20-854b-428a-940f-a2fbf057c7a0">

### SOP(Same-Origin Policy)
동일한 origin에 대해서만 resource를 공유할 수 있다.


### SOP 왜 필요할까?
SOP 정책이 없다면 다른 origin에 접근할 수 있다.

만약 다른 origin에 요청을 보낼 수 있다면 악의적으로 무한요청을 보내 웹 서버를 터뜨리는 일도 가능하다.

### 이러한 origin 비교는 브라우저가 한다.
"허용되지 않은 출처"에서 보내는 요청을 브라우저 단에서 막는다.
<img width="827" alt="image" src="https://github.com/kssumin/cs-study/assets/88534959/751b0266-5761-4063-b917-f2504398c500">

### CORS는 SOP를 해결하기 위한 정책이다.
기본적으로 브라우저는 SOP 정책에 따라 다른 origin의 접근을 차단한다.
하지만 다른 origin 간의 resource가 필요한 경우가 분명히 있다.
이를 위해 CORS 정책을 이용해 다른 origin 의 resource를 허용할 수 있다.

## 브라우저의 CORS 동작
1. client에서 HTTP 요청 header에 ```origin```을 담아 전달
2. server에서는 HTTP 응답 header에 ```Access-Control-Allow-Origin```을 담아 전달
3. 브라우저는 자신이 보낸 origin과 서버에서 응답으로 준 Access-Control-Allow-Origin을 비교한다.
   * 일치한다 : 유효한 요청이므로 resource를 가져올 수 있다.
   * 일치하지 않는다 : CORS에러 발생


## cors의 동작원리
* simple request
* preflight
* credential request

### Simple Request
"특정 조건"에서 서버에 바로 요청을 보내는 방법
<img width="767" alt="image" src="https://github.com/kssumin/cs-study/assets/88534959/b38fc7ef-ceb5-4783-a206-95797808e0ce">
서버는 header에 ```Access-Control-Allow-Origin```를 같이 보내주는데 브라우저가 이 값을 통해 CORS 정책 위반 여부를 확인한다.

#### 조건
이 3가지 조건을 모두 만족해야 한다.
* GET, HEAD 요청
* Content-Type 헤더가 다음과 같은 POST 요청
  * application/x-www-form-urlencoded
  * multipart/form-data
  * text/plain
* Accept, Accept-Language, Content-Language, Content-Type, DPR, Downlink, Save-Data, Viewport-Width, Width를 제외한 헤더를 사용하면 안된다.


사진 파일 업로드(multipart/form-data)가 아니라면 content-type은 대체로 application/json 방식으로 처리한다(REST API)

따라서 거의 대부분 preflight 방식을 사용한다.

### Preflight
지금 보내는 요청이 유효한지 확인하기 위해 OPTIONS 메서드로 예비 요청을 보낸다.
<img width="716" alt="image" src="https://github.com/kssumin/cs-study/assets/88534959/a8a8288e-4d47-46f7-bb3c-753bde2ec09b">
1. 브라우저는 서버로 HTTP OPTIONS 메서드를 통해 preflight를 먼저 보낸다.
    * origin : 요청을 보내는 주소
    * Access-Control-Request-Method : 실제 요청에 사용할 메서드
    * Access-Control-Request-Headers : 실제 요청에 사용할 헤더
2. 서버는 preflight에 대한 응답으로 허용하는 요청에 대한 헤더 정보를 담아 브라우저로 보낸다.
    * Access-Control-Allow-Origin : 허용되는 Origin의 목록
    * Access-Control-Allow-Methods : 허용되는 method들의 목록
    * Access-Control-Allow-Headers : 허용되는 header들의 목록
    * Access-Control-Max-Age : preflight 요청이 브라우저에 캐시 될 수 있는 시간(초 단위)
3. 브라우저는 서버가 보내는 정보를 통해 해당 요청이 안전한지 확인한 이후 본 요청을 보낸다.

### preflight 캐시
요청을 보내기 전에 OPTIONS 메서드로 매번 예비 요청을 보내야 한다.

하지만 API 호출 수가 늘어날 수록 예비 요청으로 인해 서버 요청이 배가 된다는 문제점이 있다.

이를 ```Access-Control-Max-Age```헤더에 캐시될 시간을 명시해주면 브라우저에 캐시를 할 수 있다.

<img width="709" alt="image" src="https://github.com/kssumin/cs-study/assets/88534959/e9ec7b77-e216-465c-9c40-6c99dcb420b6">

1. 예비 요청을 할 때마다 브라우저 캐시를 확인하여 해당 요청에 대한 응답이 있는지 확인한다.
   * 캐시가 되어있다면 : 예비 요청을 보내지 않고 바로 캐시된 응답을 사용한다.
   * 캐시가 되어있지않다면 : 예비 요청을 보내고 그 응답을 브라우저 캐시에 저장한다.

#### 왜 Simple Request를 사용하지 않고 Preflight를 사용할까?
content-length가 굉장히 큰 요청이 있다고 해보자
만약 이 요청이 CORS 정책을 위반하는 요청이라면 불필요하게 리소스를 낭비하게 된다.

이를 방지하기 위해 예비요청을 보내 이 요청이 유효한 요청인지 확인한다.

### Credential Request
클라이언트가 서버에게 자격 인증 정보(Credential)을 실어 요청할 때 사용한다.

#### Credential
Cookie, Authorization header에 설정되어 있는 토큰 값

### 해당 요청이 별도로 존재하는 이유?
기본적으로 브라우저는 쿠키와 같은 인증과 관련된 데이터를 요청할 떄 담지 않도록 설정되어 있다.

이때 요청과 관련된 정보를 담을 수 있도록 해주는 것이 ```credentials```옵션이다.

### server는 어떤 처리를 해주어야 하는가
* Access-Control-Allow-Credentials : true
* Access-Control-Allow-Origin : 와일드카드 문자("*") 사용 불가능
* Access-Control-Allow-Methods : 와일드카드 문자("*") 사용 불가능
* Access-Control-Allow-Headers  : 와일드카드 문자("*") 사용 불가능
