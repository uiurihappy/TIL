# JWT와 Session

---
## 인증과 인가의 차이
우선 인증과 인가에 대해 알아볼건데 이유는 Session과 JWT가 인증과 연관이 있으며 둘의 차이를 명확하게 알아보고자 한다.

#### 인증
내가 누군지를 판별하는 것으로 ID/PW와 같이 로그인하여 확인받는 행위라 할 수 있다.

#### 인가
허용된 권한으로 인증을 하고, 관리자인지 사용자인지 허용되는 권한을 의미한다.

---
## Session
우선 세션부터 알아볼 것이다.
**세션**은 클라이언트가 웹브라우저를 통해 웹 서버에 접속한 시점부터 웹 브라우저를 종료하는 시점까지 클라이언트가 누군지 구별하고 클라이언트에 따라 다른 권한을 주기 위해 사용되는 기술이다.

Session 정보는 DB에 저장되며 클라이언트는 SessionId를 가지고 있다. 클라이언트는 HTTP 요청 시 SessionId를 Cookie에 저장하여 요청하고 서버는 세션 조회하여 인증과 인가를 진행한다.

장점으로 사용자 정보를 굳이 Cookie에 다 저장할 필요없이 SessionId만 있으면 되기에 보안상 좀 더 좋은 방식이다.

단점으로는 서버 DB에 Session 정보를 저장하기에 DB 리소스를 잡아먹고 사용자가 많아질수록 부담이 된다.

---
## JWT
JWT(JSON Web Token)는 서로 간의 정보를 JSON 개체로 안전하게 전송하기 위한 간결하고 자체 포함된 방법을 정의하는 개방형 표준이라 한다. 이 정보는 디지털 서명되어 있으므로 확인하고 신뢰할 수 있으며 RSA 또는 ECDSA 를 사용하는 공개/개인 키 쌍을 사용하여 서명할 수 있다.

JWT의 구성은 헤더, 페이로드, 시그니처와 같은 구성으로 되어있다.

### 헤더(header)
헤더에는 alg(알고리즘)과 type(타입)이 들어있다.
해싱 알고리즘 방식 또는 토큰에 대한 타입이 저장되어 있다.


### 페이로드(payload)
페이로드에는 사용자, 토큰 정보를 나타내는 클레임(claim)이라는 것으로 구성되어 있고, key-value 형태이다. 클레임에는 사용자 마음대로 아무런 값을 넣을 수 있다. (다만 클레임이 많아질수록 토큰의 길이가 길어진다)

#### 클레임의 세 가지 종류
- registered claim: 필수는 아니지만 운영상 미리 정의된 클레임이다. iss(발급자), exp(만료 시간), sub(제목), aud(대상) 등등 이루어져 있습니다.

- public claim: JWT를 사용하는 사용자들이 마음대로 정의할 수 있다. 충돌을 방지하기 위해 IANA JSON Web Token Registry에서 정의하거나 충돌 방지 네임스페이스를 포함하는 URI로 정의해야 한다고 한다.

- private claim: 등록되어 있지도, 공개되지도 않은 클레임이다.
 
### 시그니처(signature)
시그니처는 인코딩 된 헤더, 페이로드 그리고 비밀키(secret_key)를 가지고 헤더에 정의된 알고리즘으로 해싱하여 생성한다.

헤더와 페이로드는 암호화된 값이 아니라 base64로 인코딩 된 값이기 때문에 위 이미지처럼 누구나 확인이 가능하다. 그렇기 때문에 토큰을 검증하는 서명(signature)이 정말 중요하며 서명에 사용되는 secret_key를 잘 관리해야 한다.


서버에서는 특정 알고리즘을 통해 서명(Signature)되어 생성된 문자열을 클라이언트로 전달하고 클라이언트는 이 jwt token을 가지고 있다가 api를 호출할 때 전달해주기만 하면 된다. 전달된 jwt token은 서버에서 decoding 하여 payload를 확인한 다음 권한을 부여하게 됩니다.

---
### 세션과 비교 시 장단점
jwt는 DB에 저장하는  방식이 아니기 때문에 DB 리소스가 필요없다는 장점이 있다. 또한 여러 서비스를 운영할 때 jwt 토큰을 사용하면 session처럼 특정 sesson DB에 접근하는 게 아니라 jwt 토큰 검증 로직을 통해 여러 서비스 간에 통신에서 권한을 쉽게 제한하고 허가할 수 있다는 장점도 있다.

단점으로는 jwt 토큰이 탈취되었을 경우 누구나 해당 토큰의 권한을 가지고 서비스를 이용할 수 있다. 그리고 누구나 헤더와 페이로드의 내용을 볼 수 있기 때문에 보안에 위협이 될만한 정보나 개인정보가 있다면 노출될 수 있다는 단점도 있을 수 있다. 이러한 이유로 보통 jwt토큰의 만료시간(exp)을 짧게 정하고 리프레시 토큰(Refresh Token)과 함께 사용한다.

또 다른 단점으로는 토큰에 담고있는 정보가 많아질수록 데이터의 크기가 커진다. 그렇게 되면 네트워크 전달 시 데이터의 크기로 부하가 생길 수 있다.

리프레시 토큰은 보통 Inmomry-DB(Redis)에 저장되며 긴 만료시간을 가지고 있다. 사용자는 jwt 토큰이 만료되면 리프레시 토큰을 이용해 갱신을 하여 사용하게 된다. 우리가 앱에서 별도의 재로그인 없이 계속 서비스를 이용할 수 있는 것은 리프레시 토큰으로 재갱신을 해주었기 때문이다.
