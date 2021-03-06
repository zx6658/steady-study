---
title: 기본 인증, 다이제스트 인증
createdDate: "2019-11-23"
updatedDate: "2019-11-23"
author: Ideveloper
tags:
  - network
image: authorization.png
draft: false
---

`이 글은 HTTP 완벽가이드 12, 13장을 참고해 정리한 포스팅입니다.`

## 시작하기에 앞서

- **모든 정보나 업무가 공용은 아니기 때문에, 허가된 사람만이 데이터에 접근하고 업무를 처리할 수 있어야 합니다.**

- **그러기 위해서는 서버가 사용자가 누구인지 식별할 수 있어야하고, 인증은 본인이 누구인지 증명을 하는것입니다. 이러한 인증관련 기능을 HTTP에서 자체적으로 기능을 제공하고 있습니다.**

- **이 글에서는 HTTP 인증(기본이 되는 기본인증, 다이제스트 인증)에 대해서 알아볼것입니다.**

---

## 인증

- 인증은 본인이 누구인지 증명하는 과정이고, 비밀번호는 누군가가 추측하거나 엿들을 수 있고, 신분증은 도둑맞거나 위조 될 수 있습니다. 하지만 인증에 쓰이는 데이터들은 누구인지 판단하는데 도움이 됩니다.

#### HTTP의 인증요구/응답

- HTTP는 필요에 따라 고쳐 쓸 수 있는 제어 헤더를 통해, 다른 인증 프로토콜에 맞게 확장할수 있는 프레임워크를 제공합니다.
- HTTP에는 `기본 인증`과 `다이제스트 인증`이라는 두가지 공식적인 인증 프로토콜이 있습니다.

표 - 네가지 인증단계

| 단계      |        헤더        |                                              설명                                              |      메서드/상태 |
| :-------- | :----------------: | :--------------------------------------------------------------------------------------------: | ---------------: |
| 요청      |                    |                                   첫 번째 요청시 인증정보 x                                    |              GET |
| 인증 요구 |  WWW-Authenticate  | 서버는 사용자이름과 비밀번호를 제공하라는 의미의 401 상태정보와 함께 요청 반려, 인증 영역 표시 | 401 Unauthorized |
| 인증      |   Authorization    |                 인증 알고리즘과 사용자 이름과 비밀번호를 기술한 정보 함께 보냄                 |              GET |
| 성공      | Authorization-Info |                   인증정보 확인 후 응답반환, 인증 세션에 관한 추가정보 기술                    |           200 OK |

보안영역

- HTTP는 각 리소스마다 다른 접근 조건을 다룰수 있는데, 서버가 인증요구를 할때, **realm** 지시자를 명시함으로써 가능해집니다.
- 따라서, realm에 "executive-committee@company.com" 같은 서버의 호스트명을 넣는것도 유용할 수 있습니다.

```
HTTP1.0 401 Unauthorized
WWW-Authenticate: Basic realm="About"
```

![image](https://user-images.githubusercontent.com/26598542/69476541-81572080-0e1e-11ea-82e7-b1db48d0f979.png)

---

## 기본인증

- 기본 인증은 원래 HTTP/1.0에 기술되어 있었지만, HTTP 인증의 상세 내용을 다루는 RFC 2617로 옮겨졌습니다.

**아래는 기본인증과정에 대한 그림 예시입니다.**

![image](https://user-images.githubusercontent.com/26598542/69476371-7b604000-0e1c-11ea-8020-14bd69d8db2c.png)

- (a) about.jpg 정보 요청
- (b) 서버는 401 Unauthorized 응답과 함께 WWW-Authenticate 헤더를 기술해 어디서 어떻게 인증할지 설명
- (c) 인코딩된 비밀번호와 그 외 인증 파라미터들을 Authorization 헤더에 담아 요청 보냄, 사용자 이름과 비밀번호를 입력해 세미클론으로 이어 붙여 `Base64 인코딩` 방식으로 인코딩해 같이 보내줌
  - `Base64`로 인코딩하는 이유는 전송 중 원본 문자열이 변질될 걱정이 없고, (큰따옴표, 콜론, 캐리지 리턴)과 같은 HTTP헤더에서 사용할수 없는 정보를 보낼때도 유용하기 때문
  - _Base 64 인코딩에 대한 설명 링크_ (https://ifuwanna.tistory.com/72)
- (d) 인증이 완료되면, 상태코드와 함께 응답 반환 (추가적인 인증 알고리즘에 대한 정보를 Authenticate 헤더에 기술할수도 있음)

**프락시 인증**

- 사용자들이 회사의 서버나 LAN,무선 네트워크에 접근하기 전에 프락시 서버를 거치게 하여 사용자를 인증할때 `프락시 인증`을 사용하게 됩니다.
- 회사 전체에 대해 통합적인 접근 제어를 하기 위해 프락시 서버를 사용하면 좋습니다.
- 프락시 인증은 웹서버의 인증과 헤더와 상태코드만 다르고 절차는 같습니다.

표 - 웹 서버 인증 vs 프락시 인증

|        웹서버        |               프락시 서버 |
| :------------------: | ------------------------: |
| 비인증 상태코드: 401 |      비인증 상태코드: 407 |
|   WWW-Authenticate   |        Proxy-Authenticate |
|    Authorization     |       Proxy-Authorization |
| Authentication-info  | Proxy-Authentication-info |

---

## 기본 인증의 보안 결함

- 기본인증은 악의적이지 않은 누군가가 의도치 않게 리소스에 접근하는 것을 막는데 사용하거나, SSL같은 암호기술과 혼용합니다.

**기본인증이 가지는 보안결함은 아래와 같습니다.**

- base-64인코딩은 어렵지 않게 디코딩 할 수 있습니다.

  - 따라서, HTTP 트랜잭션을 SSL암호화 채널을 통해 보내거나, 다이제스트인증 같은 프로토콜을 사용하는 것이 좋습니다.

- 보안 비밀번호가 디코딩하기에 복잡한 방식으로 인코딩되어 있다고 하더라도, 제사자는 사용자 이름과 비밀번호를 캡쳐해 원 서버에 보내 인증에 성공해 서버에 접근 할수 있습니다.

  - 이는, 기본인증은 재전송 공격을 예방하기 위한 어떤 일도 하지 않기 때문입니다.

- 기본 인증이 보안이 뚫리더라도 치명적이지 않은 애플리케이션에 사용된다 하더라도, 동일한 정보로 주요 온라인 은행사이트 같은 곳에 접근할수도 있습니다.

- 트랜잭션에 본래의도를 바꿔버리는 프락시나 중개자가 중간에 개입하는 경우, 기본 인증은 정상적인 동작을 보장하지 않습니다.

- 가짜 서버의 위장에 취약합니다.

**종합해보면, 기본인증은 일반적인 환경에서 개인화나 접근을 제어하는데 편리하며, 다른 사람이 보지 않기를 원하기는 하지만, 보더라도 치명적이지 않은 경우에는 여전히 유용합니다.**

---

## 다이제스트 인증

- 기본 인증은 편하고 유연하지만 안전하지 않습니다. 인증정보를 평문으로 보내고, 메시지를 위조하지 못하게 하려는 어떠한 시도도 하지 않습니다.
- 다이제스트 인증은 기본 인증과 호환되는 더 안전한 대체재로서 개발되었습니다.

### 다이제스트 인증의 개선점

- 비밀번호를 절대로 네트워크를 통해 평문으로 전송하지 않는다.
- 인증 체결을 가로채서 재현하려는 악의적인 사람들을 차단한다.
- 구현하기에 따라서, 메시지 내용 위조를 막는 것도 가능하다.
- 그 외 몇몇 잘 알려진 형태의 공격을 막는다.

**위와 같은 개선점들이 있긴하지만, 다이제스트 인증 역시도 요청과 응답의 나머지 부분에 대해서는 누군가가 엿보는 것이 가능하므로, TLS나 HTTP가 더 안전한 프로토콜입니다.**

#### 비밀번호를 안전하게 지키기 위한 "요약"

- 비밀번호를 보내는 대신 클라이언트는 비밀번호를 이전 상태로 되돌아갈수 없는 상태로 뒤섞은 fingerprint 혹은 digest(요약)을 보내게 됩니다.
- 위와 같이 보냄으로써, 클라이언트와 서버는 둘다 비밀번호를 알고 있으므로, 서버는 요약을 검사할수 있게 됩니다.
- 이러한 요약은 제 3자가 알아내기 어렵습니다.

#### 단방향 요약

- 요약은 정보 본문의 압축이고, 단방향 함수로 동작합니다.
  - 단방향 알고리즘은 암호화는 수행하지만 절대로 복호화가 불가능한 알고리즘을 뜻합니다.
- 대표적인 요약 함수는 MD5이고, 임의의 바이트 배열을 원래 길이와 상관없이 128비트 요약으로 변환합니다.(32글자의 16진수 문자로 표현됨)
- `요약 함수`는 보통 암호 체크섬으로 불리며 단방향 해시함수이거나 지문함수입니다.

#### 재전송 방지를 위한 난스(nonce) 사용

- 비밀번호를 요약하는 것만으로도 위험한 경우가 있는데, 이러한 요약을 가로채 서버에 몇번이고 재전송할수 있기 떄문입니다.
- 이러한 재전송 공격을 방지하기 위해 서버는 클라이언트에게 난스라고 불리는 특별한, 그리고 자주 바뀌는 증표를 건네주게 됩니다.
  - IBM Knowledge center 사이트에 설명되어 있는 nonce : https://www.ibm.com/support/knowledgecenter/ko/SSAW57_9.0.5/com.ibm.websphere.nd.multiplatform.doc/ae/cwbs_nonce.html
- 난스를 비밀번호에 섞으면 난스가 바뀔때마다 요약도 바뀌게 만들어주고, 특정 난스 값에 대해서만 요약이 유효해 짐으로써, 재전송 방지를 하게 됩니다.
- 난스는 WWW-Authenticate 인증 요구에 담겨 서버에서 클라이언트로 넘겨지게 됩니다.

#### 다이제스트 인증 핸드셰이크

- 기본인증에서 사용하는것과 비슷한 헤더를 사용하는 강화된 버전이고 선택적 헤더인 Authorization-Info가 새롭게 추가되었습니다.

**다이제스트 인증이 이루어지는 과정을 그림으로 요약하면 아래와 같습니다.**
![image](https://user-images.githubusercontent.com/26598542/69477481-2a574880-0e2a-11ea-96be-ebda9de3e306.png)

- 위 과정에서 클라이언트가 난스를 보낼때도 있는데, 이는 클라이언트가 서버를 인증하길 원할때 입니다.

### 다이제스트 인증의 요약 계산

- 다이제스트 인증의 핵심은 공개된 정보, 비밀정보, 난스 값을 조합한 단방향 요약입니다.

**A1.A2 두 조각의 데이터는 요약을 생성하기 위해 H와 KD에 의해 처리됩니다.**

- 단방향 해시 함수 H(d)와 요약함수 KD(s,d) . 여기서 s는 비밀(secret)을, d는 데이터를 의미합니다.
  - 요약 알고리즘은 여러가지를 선택할수 있도록 지원하며 MD5와 MD5-sess가 RFC-2617에서 제안된 알고리즘입니다.
- 비밀번호 등 보안 정보를 담고 있는 보안 정보를 담고 있는 데이터 덩어리, A1이라 칭합니다.
- 요청 메시지의 비밀이 아닌 속성을 담고있는 데이터 덩어리 A2라 칭합니다.
  - **RFC 2617은 `quality of protection(qop)`에 따른 A2의 두가지 사용법을 정의하고 있습니다.**
    - qop="auth" (HTTP요청메서드와 URL만 포함)
    - qop="auth-int" (메시지 무결성 검사를 위한 메시지 엔터티 본문 추가)

### 다이제스트 인증 세션

- 인증 세션은 클라이언트가 다른 서버로부터 또 다른 WWW-Authenticate 인증 요구를 받을때까지 지속됩니다.
- 다이제스트 인증에서는 난스가 만료되면 클라이언트가 다시 요청을 보내도록 서버는 새 난스 값과 함께 401 응답을 반환할 수 있고, 이때 `stale="true"` 를 명시해 줌으로써 사용자 이름 비밀번호를 다시 입력받을 필요없이 새 난스값으로 요청을 보내도록 지시해 줄 수 있습니다.

### 사전(preemptive)인가

**기본인증에서의 사전인가**

- 사전 인가는 `기본인증`에서는 사소하고 흔한 일이다.
  - 브라우저에서 사용자 이름과 비밀번호 들에 대한 클라이언트 측 데이터베이스를 관리
  - 한번 인증을 하면, 그 URL에 대한 다음번 요청때 올바른 Authorization 헤더를 전송

**다이제스트 인증에서의 사전인가**

- `다이제스트 인증`에서는 조금 복잡한데, 인증요구를 받기전까지는 난스 정보가 없기 때문에 올바른 Authorization 헤더 정보를 알수 없습니다.
- 다이제스트 인증에서 사전인가를 하는 방법은 아래와 같습니다.
  - 다음 난스를 서버가 Authentication-Info 성공 헤더에 담아 미리 보냄
  - 짧은 시간동안 같은 난스를 재사용 하는것 허용
  - 클라이언트와 서버가 동기화되고 예측가능한 난스 생성 알고리즘 사용

### 기본인증과 다이제스트 인증의 HTTP인증 헤더 비교

|   단계    |                 기본 인증                  |                                                                                    다이제스트 인증 |
| :-------: | :----------------------------------------: | -------------------------------------------------------------------------------------------------: |
| 인증 요구 |  WWW-Authenticate: Basic realm="<영역값>"  |                                        WWW-Authenticate: Digest realm="<영역값>" nonce="<난스 값>" |
|   응답    | Authorization: Basic "<base64(user:pass)>" | Authorization: Digest realm="<영역값>" nonce="<난스 값>" username="<사용자 이름>" uri="<요청 uri>" |
|   정보    |                    없음                    |                                                        Authentication-Info: next-nonce="<난스 값>" |

### 실제 상황에 대한 고려

다이제스트 인증 작업을 할때 고려해야할 것이 몇가지 있는데 아래와 같습니다.

- **다중 인증요구**
  - 서버가 클라이언트의 능력을 모른다면, 기본 및 다이제스트 인증 요구를 모두 보낼 것입니다. 이때 클라이언트는 자신이 지원할 수 있는 가장 강력한 인증 매커니즘을 선택해야 합니다.
  - 사용자의 에이전트는 `WWW-Authenticate` 나 `Proxy-Authenticate`헤더 필드 값을 분석 할때 주의를 기울여야 하는데, 헤더들에 인증요구가 둘이상 포함되거나 WWW-Authenticate 헤더가 둘 이상 제공될수 있기 때문입니다.
- **오류처리**
  - 다이제스트인증에서, 지시자나 그 값이 적절하지 않거나 요구된 지시자가 빠져 있는 경우 알맞은 응답은 `400 Bad Request` 입니다.
  - 요청의 요약이 맞지않거나, uri지시자가 가리키는 리소스가 요청줄에 명시된 리소스와 다르면 공격의 징후일 수 있습니다.
- **보호공간 (Protection Space)**
  - 기본인증에서 클라이언트는 요청 URI와 그 하위의 모든 경로는 같은 보호 공간에 있는 것으로 가정합니다.
  - 다이제스트 인증에서, 인증요구의 `WWW-Authenticate: domain` 필드는 보호 공간을 보다 엄밀하게 정의합니다.
  - domain 필드는 작은 따옴표로 묶인 URI의 공백으로 분리된 목록이고, 목록의 모든 url와 그 하위에 위치한 uri는 같은 보호 공간에 있는것으로 가정합니다.
  - domain 필드가 없거나 빈값이면, 인증을 요구하는 서버의 모든 URI는 그 보호공간에 있는 것입니다.
- **URI 다시쓰기**

  - 프락시는 아래 예와 같이 리소스의 변경 없이 구문만 고쳐서 URI를 다시 쓰기도 합니다.
    또한, 프락시가 uri를 변경할수 있는 동시에 다이제스트 인증은 URI 값의 무결성을 검사하므로, 다이제스트 인증은 이러한 변경에 의해 실패 할 수 있습니다.

  ```
  호스트 명은 정규화되거나 IP 주소로 대체될 수 있습니다.
  문자들은 "%" escape 형식으로 대체될수 있습니다.
  특정 원 서버로부터 가져오는 리소스에 영향을 주지 않는, 타입에 대한 추가속성이 삽입될수 있습니다.
  ```

- **캐시**

  - 어떤 공유 캐시가 Authorization 헤더를 포함한 요청과 그에 대한 응답을 받은 경우, 다음 두 Cache-Control 지시자 중 하나가 응답에 존재하지 않는 한 다른 요청에 대해 그 응답을 반환해서는 안됩니다.

  ```
  원 서버의 응답이 "must-revalidate" Cache-Control 지시자를 포함한 경우 (요청헤더 이용 재검사 수행)
  원 서버의 응답이 "public" Cache-Control 지시자를 포함한 경우
  ```

### 보안에 대한 고려 사항

- 헤더 부당 변경
  - 양 종단 암호화 헤더에 대한 디지털 서명 필요
  - 다이제스트 인증은 보호를 데이터 까지 확장하는 것은 X (보호 수준에 대한 정보는 WWW-Authenticate, Authorization 헤더에 담겨 있음)
- 재전송 공격
  - 재전송 공격이란 누군가 트랜잭션에서 엿들은 인증 자격을 다른 트랜잭션을 위해 사용하는것을 말합니다.
  - 문제를 완화하는 방법은 `클라이언트의 IP주소`, `타임스탬프`, `리소스의 ETag`, `개인서버 키` 에 대한 요약을 포함하는 난스를 서버가 생성하도록 하는것입니다.
  - 완전히 피할수 있는 방법은 매 트랜잭션마다 유일한 난스 값을 사용하는것입니다.
- 다중 인증 메커니즘
  - 클라이언트에게 가장 강력한 인증 메커니즘을 선택해야 할 의무가 있는 것은 아닙니다.
  - 위 문제를 피하기 위해서는 강력한 인증제도를 유지하는 프락시 서버를 사용하는 것인데, 사내 네트워크 같은 모든 클라이언트가 인증제도를 지원할 수 있는 환경일때 가능합니다.
- 사전 공격
  - 전형적인 비밀번호 추측 공격입니다.
  - 실질적인 해결 방법은 복잡한 비밀번호 사용, 혹은 괜찮은 비밀번호 만료 정책 이외에는 없습니다.
- 악의적인 프락시와 중간자 공격
  - 많은 인터넷 트래픽이 오늘날 한 프락시에서 다른 프락시로 이동합니다. 만약 이들 프락시중 하나가 악의적이거나 보안이 허술하다면, 클라이언트는 중간자 공격에 취약한 상태가 될 가능성이 있습니다.
  - 해결할 유일한 방법은 SSL을 사용하는 것입니다.
- 선택 평문 공격

  - 악의적인 프락시가 트래픽 중간에 끼어든다면 어렵지 않게 클라이언트가 응답 계산을 하기 위한 난스를 제공하게 되고, 응답 계산을 위해 알려진 키를 사용하는 것은 응답의 암호 해독을 쉽게 합니다. 이것을 선택 평문 공격이라 합니다. 그 예는 아래와 같습니다.

  ```
  미리 계산된 사전 공격
  자동화된 무차별 대입 공격
  ```

  - 방지 방법은 서버에서 제공한 난스 대신 선택적 난스 지시자를 사용하여 응답을 생성하고, 강력한 비밀번호 및 만료 정책을 세우는 것입니다.

- 비밀번호 저장
  - 다이제스트 인증 비밀번호 파일이 유출되면, 영역의 모든 문서는 즉각 공격자에게 노출됩니다.
  - 여전히 콘텐츠에 대한 보안 측면에서는 어떠한 보호도 제공하지 못하므로, 진정한 보안 트랜잭션은 SSL을 통해서만 가능합니다.
