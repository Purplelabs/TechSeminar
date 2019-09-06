# CORS는 왜 필요한가

Cross-Origin Resource Sharing에 대한 원리와 필요한 이유에 대해 다룬다.

## 월드 와이드 웹

인터넷에 연결된 컴퓨터를 통해 정보를 공유할수 있는 정보공간이며 흔히 Web이라고 부른다. Web은 인터넷을 이용해 정보를 공유하는 하나의 서비스 개념으로 볼수있음. 정보를 쉽게 공유하기 위해서 아래와 같은 형식을 일반적으로 사용한다.

- HTTP : 웹에서 정보를 주고 받기 위한 통신 프로토콜
- HTML : 웹에서 브라우저를 통해 페이지를 보여줄수 있는 마크업 언어
- 하이퍼링크 : HTML문서안에서 다른 자료로 연결을 하고 가리킬수 있는 기능. 주로 URL을 가리킨다.

### URL

Uniform Resource Locator의 약자로써 리소스를 가리키기 위한 텍스트로 구성된 웹의 주소

``` link
https://www.purplelabs.co.kr
https://www.purplelabs.co.kr/index.html
https://www.purplelabs.co.kr/index.html?query=value
```

다음과 같은 구성요소를 갖고있다.

| Scheme or protocol | Host | Port | Path | Query | Fragment |
| ------------------ | ---- | ---- | ---- | ----- | -------- |
| https://|purplelabs.co.kr | :80 | purplelabs.html | ?key=value | #SomewhereInTheDocument |

- Scheme or protocol : url에서 사용하는 프로토콜이나 스키마를 지정. 프로토콜은 http:// 형식을 사용하며 스키마는 mailto:와 같은 형식을 사용한다.
- Host : 도메인이나 ip등 url에서 가리키는 주소
- Port : URL이 가리키는 주소의 port. http는 기본이 80이라 생략하면 80 port를 사용
- Path : 주소에서 가리키는 리소스의 경로
- Query : 주소에서 특별한 값을 보내고 싶을때 사용
- Fragment : 리소스 내에서 스스로 특정한 위치를 가리키는 영역

### Origin

웹에서의 오리진은 URL로 특정 개체를 가리킬때 구성요소인 Scheme or protocol, Host, Port가 같은지 판단하는 요소이다. CORS에서 판단의 근거가 됨.
``` link
# protocol, host, port가 같이때문에 same origin
https://www.purplelabs.co.kr/purplelabs.html
https://www.purplelabs.co.kr/purplelabs-healthcare.html

# host가 달라 same origin이 아님
http://example.com
http://www.example.com
http://myapp.example.com
```

### Same Origin Policy (SOP)

문서 나 스크립트가 다른 origin 인 정보와 통신하는걸 제한하는 규약. 대부분의 modern browser가 도입하고 있음. 모든 경우에 제한을 걸어버리면 사이트간 하이퍼링크가 금지되기 때문에 모든 요소에 걸려있지 않으며 대부분 문서 내의 포함되어있는(html 내부) 요소들은 가능하지만 script등으로 읽어낼때는 제한을 둔다.

#### Same Origin Policy 정책을 따르지 않는다면 나올수 있는 보안 취약점들

- Clickjacking : 악의적인 사이트에서 iframe으로 특정 사이트를 띄워놓고 버튼위에 투명한 버튼등을 씌워 원하지 않는 사이트로 이동하거나 데이터를 획득
![clickjacking](https://web.dev/same-origin-policy/clickjacking.png)

- XSS (Cross Site Scripting) : 공격자가 게시판등 다수의 사람이 작성한 페이지가 노출되는 사이트에서 악성 스크립트를 노출 시키고 스크립트로 피해자의 개인정보들을 공격자의 서버로 보냄

### web resource

- web page는 모든 리소스를 http를 통해 데이터 전달
- 리소스들은 보안상의 이유로 브라우저에서 Same-origin policy를 적용함
- 웹 사이트는 개발 편의성을 위해 Same-origin policy를 완화할 필요가 있음

## Cross-Origin Resource Sharing (CORS)

Same-origin policy룰 완화하기 위해 동적으로 신뢰성이 있는(api server) 서버의 리소스를 접근할때 서버측에서 안전하다고 알려주는 기술. 초기엔 W3C에서 권고안이었으나 현재는 obsolete 상태이며 WHATWG 에서 정의한 javascript의 fetch api와 함께 정의하고 있다.

현재의 정의로는 fetch를 통한 HTML의 form 이상의 요소들을 가져오기 위해 외부에서 들어오는 응답을 허용해주기 위한 프로토콜로 지정되어있다.

- W3C(World Wide Web Consortium) : 1994년 설립한 월드 와이드 웹의 표준을 위한 협회. 웹 표준과 가이드라인을 제시함
- WHATWG(Web Hypertext Application Technology Working Group) : 2004년에 애플, 모질라, 오페라에서 설립하여 현재 구글, 마이크로소프트도 함께 주도하고 있는 웹 표준화 조직

2018년 4월부터 각각 웹 표준을 제시하던 두 조직이 하나의 표준을 제시하기로 했으며 WHATWG가 주도권을 가져감. [관련기사](https://www.zdnet.co.kr/view/?no=20190531184644)

### AsynchronousJavaScriptAndXML(AJAX)

자바스크립트에서 비동기로 외부 요청을 할때 쓰이는 기술로 대표적인 Api는 두가지가 있다.

- XMLHttpRequest : IE 때부터 사용하던 api로 현재는 사용이 지양되고있다. 내장함수와 콜백이 구현하기 복잡하게 되어있음.
- Fetch : XMLHttpRequest를 대체하기 위해 ES6 부터 표준화된 api. Promise 기반인것이 특징.

### CORS를 사용하는 요청들

- 외부 도메인으로의 XMLHttpRequest나 Fetch API 호출
  - ajax로 들어온 모든 리소스(json, image등)은 script 내부에서 동작하기 때문에 외부 리소스로 간주함.
- CSS @font-face를 통한 Cross-domain 폰트 사용
- WebGL 텍스쳐
- drawImage를 사용해 캔버스에 드로잉되는 이미지/비디오 프레임들.

### 기본적인 작동 원리

브라우저는 Same Origin Policy정책에서 비허용된 요청에 Origin 헤더를 붙이고 응답 헤더에 Access-Control-Allow-Origin에 * 혹은 요청한 Origin 헤더와 같은지를 판단해 데이터를 신뢰한다.

![cors](https://cloud.ibm.com/docs-content/v1/content/e7fa28afca5ed5853f22f939e8c1142fe9af4814/nl/ko/infrastructure/CDN//images/cors-simple.png)

### Origin Header

CORS가 필요한 요청과 HEAD와 GET을 제외한 요청에는 요청 헤더에 Origin을 포함한다.

### Preflight request

CORS protocol을 제대로 이해하고 있는 서버인지 검증하는 user agent 구현. OPTIONS 메소드를 통해 사전에 가능한 Methods인지 확인하고 확인한 메소드를 보냄. 브라우저 수준에서 구현되어 있어 클라이언트 개발자가 구현해야 하지는 않음.

### CORS와 XSS의 관계

XSS는 악의적인 스크립트를 다른 사이트에서 실행되게 하는게 주 목적이며 CORS는 SOP를 완화시키기 위한 기술이기 때문에 직접적으로 관련이 없다. 이미 악의적인 스크립트를 실행하는 순간에는 해커가 설정한 서버에서 CORS허용등이 가능해서 CORS측면에서는 보안상 막을 수 없지만 다른 외부 사이트로의 요청은 보낼 수 없기때문에 XSS가 취약한 사이트서는 어느정도 제한은 가능하다.
