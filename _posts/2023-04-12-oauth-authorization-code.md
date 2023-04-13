---
title: (OAuth2.0) Authorization Code Grant (권한 부여 인증 방식)
author: jun
date: 2023-04-12 19:31:00 +0900
categories: [Web, Auth]
tags: [oauth]
comments: true
img_path: /assets/img/post/20230412/
---

## OAuth 2.0

OAuth2.0은 **인증에 대한 표준 프로토콜**입니다. 이 프로토콜은 여러 디바이스에서 특정 서비스에 대한 인증을 간편하게 할 수 있도록 설계되어 있습니다.

카카오, 구글, 페이스북 등에서 제공하는 **간편 로그인 기능**도 해당 프로토콜을 따라 구현되어 있습니다. 

> 본 포스트에서는 OAuth2.0 권한 부여 방식에서 가장 많이 사용되는 Authorization Code Grant 방식에 대해서 기술합니다.

### OAuth Access Token

Access Token은 리소스서버로 요청하기 위해 클라이언트에서 사용하는 토큰을 의미합니다. 

여기서 리소스서버는 **요청에 대한 자원을 리턴해주는 서버**이며, 클라이언트는 **요청을 하는 주체**(예를 들면, 우리가 만든 Application)를 의미합니다.

> ID Token vs Access Token
>
> Access Token은 OAuth2.0 프토콜에서 정의되고, ID Token은 OAuth2.0의 **확장 프로토콜**인 OpenID Connect에서 정의됩니다.
>
> 둘의 가장 큰 차이점은, Access Token은 리소스서버에서 읽고 유효성을 검사하며 **JWT일수도 있고 임의의 문자열**일수도 있습니다. 즉, **API를 통해 리소스서버로부터 자원을 리턴받는 용도로만 사용**됩니다.
>
> **ID Token은 JWT**입니다. 이 토큰에는 사용자(Resource Owner)의 이름이나 이메일 주소와같은 정보가 포함될 수 있으며, **해당 토큰을 활용해 각 서비스에서는 인증기능을 구현**할 수 있습니다. (서비스 세션 대신 ID Token을 활용하여 SSO 구현 가능)

## OAuth 2.0 Authorization Code Grant

인증 코드 권한 부여 방식은 클라이언트에서 인증코드(Authorization Code)로 인증 서버로부터 Access Token을 받는 방식 입니다.

### Authorization Code Flow (흐름)

인증코드로 엑세스 토큰을 받기위한 큰 흐름은 다음과 같습니다.

1. 애플리케이션이 브라우저를 열어 사용자를 OAuth 서버로 보냅니다. (카카오, 구글 등에서 제공하는 로그인 화면)
2. 사용자는 인증 프롬프트를 보고 앱의 요청을 승인합니다. (로그인)
3. 사용자는 쿼리 파라미터에 인증 코드가 있는 애플리케이션으로 다시 리디렉션됩니다. (callback)
4. 애플리케이션이 인증 코드를 액세스 토큰으로 교환합니다.

큰 흐름을 자세하게 살펴보겠습니다.

### 사용자의 허가 받기 (1~2) 

OAuth는 사용자가 애플리케이션에 대한 제한된 액세스 권한을 부여할 수 있도록 하는 것입니다. 

애플리케이션은 먼저 요청하는 권한을 결정한 다음, 사용자를 브라우저로 보내 **권한을 얻도록** 해야 합니다. 인증 flow를 시작하기 위해 애플리케이션은 다음과 같은 URL을 구성하고 해당 URL에 대한 브라우저를 띄워줍니다.

```http
https://authorization-server.com/auth
 ?response_type=code
 &client_id=29352915982374239857
 &redirect_uri=https%3A%2F%2Fexample-app.com%2Fcallback
 &scope=create+delete
 &state=xcoiv98y2kd22vusuye3kch
```

GET 파라미터에 대한 설명은 다음과 같습니다.

- `response_type=code` - 애플리케이션이 Authorization Code Grant 방식임을 인증 서버에 알립니다.
- `client_id` - 개발자가 애플리케이션을 처음 등록할 때 얻은 애플리케이션의 식별자입니다.
- `redirect_uri` - 사용자가 요청을 승인한 후 사용자를 다시 보낼 위치를 인증 서버에 알려줍니다.
- `scope` - 응용 프로그램이 요청하는 권한을 나타내는 하나 이상의 공백으로 구분된 문자열입니다. 사용 중인 특정 OAuth API는 지원하는 범위를 정의합니다.
- `state` - 애플리케이션이 임의의 문자열을 생성하여 요청에 포함합니다. 사용자가 앱을 승인한 후 동일한 값이 반환되는지 확인해야 합니다. 이는 [CSRF 공격을](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) 방지하는 데 사용됩니다.

사용자가 이 URL을 방문하면 인증 서버는 이 애플리케이션의 요청을 인증할 것인지 묻는 메시지를 표시합니다.

![image-20230402223750140](/image-20230402223750140.png){: w="400"}
_카카오 로그인 화면_

### Application으로 리디렉션 (3)

사용자가 요청을 승인하면 인증서버는 예를들어, 다음과 같은 URI로 리디렉션 시킵니다. (인증서버에서 애플리케이션으로 리디렉션)

```http
https://example-app.com/redirect
 ?code=g0ZGZmNjVmOWIjNTk2NTk4ZTYyZGI3
 &state=xcoiv98y2kd22vusuye3kch
```

`state`값은 애플리케이션이 처음 요청에 설정한 값과 동일합니다. 

애플리케이션은 인증 코드를 요청할 때 설정한 state값과 GET 파라미터로 넘어온 state값이 일치하는지 확인해야 합니다. 이는 CSRF 및 기타 관련 공격으로부터 보호합니다.

`code`는 인증 서버에서 생성한 인증 코드입니다 . 이 코드는 상대적으로 수명이 짧으며 일반적으로 OAuth 서비스에 따라 1~10분 동안 지속됩니다.

### Authorization Code로 Access Token 얻기 (4)

애플리케이션은 다음 파라미터를 사용하여 서비스의 토큰 엔드포인트에 대한 POST 요청을 만듭니다.

- `grant_type=authorization_code` - 애플리케이션이 Authorization Code Grant 방식을 사용하고 있음을 토큰 엔드포인트에 알려줍니다.
- `code` - 인증서버로 부터 부여받은 인증코드 입니다.
- `redirect_uri` - 코드를 요청할 때 사용된 것과 동일한 리디렉션 URI입니다.
- `client_id` - 애플리케이션의 클라이언트 ID 입니다.
- `client_secret` - 애플리케이션의 클라이언트 secret 입니다. 이 값을 포함하면, 액세스 토큰을 가져오기 위한 요청이 인증 코드를 가로챈 특정 공격자가 아닌 애플리케이션에서만 이루어집니다.

Access Token을 얻기위한 예제코드는 [이곳](https://github.com/sjsage522/member/blob/main/src/main/java/com/junseok/member/oauth/OAuth2LoginService.java#L89)에서 확인하실 수 있습니다.

인증서버의 엔드포인트에서는 BODY 데이터들을 확인하여 코드가 만료되지 않았는지, 클라이언트 ID와 secret이 일치하는지 확인합니다. 모든 것이 확인되면 액세스 토큰을 반환합니다.

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"MTQ0NjJkZmQ5OTM2NDE1ZTZjNGZmZjI3",
  "token_type":"bearer",
  "expires_in":3600,
  "refresh_token":"IwOGYzYTlmM2YxOTQ5MGE3YmNmMDFkNTVk",
  "scope":"create delete"
}
```

Access Token을 bearer 헤더에 포함하여 리소스 서버로 API 요청을 할 수 있게 됩니다.

만약 인증서버에서 OAuth 2.0 확장 프로토콜인 OpenID Connect을 지원한다면, 응답에 ID Token이 포함됩니다.

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"MTQ0NjJkZmQ5OTM2NDE1ZTZjNGZmZjI3",
  "id_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ"
  "token_type":"bearer",
  "expires_in":3600,
  "refresh_token":"IwOGYzYTlmM2YxOTQ5MGE3YmNmMDFkNTVk",
  "scope":"create delete"
}
```

ID Token은 JWT이며 해당 토큰에는 scope에 따라, 사용자 인증 정보들이 포함될 수 있습니다. 애플리케이션에서는 서비스 세션 대신 ID Token을 사용하여 통합 인증(Single Sign-On, SSO)을 구현할 수도 있습니다.

## 마침

> 궁금하시거나, 부족한 내용이 있다면 코멘트 부탁드립니다.  🙇🏻‍♂️

## 레퍼런스

OAUTH Authorization Code Grant Flow<br/>[https://developer.okta.com/blog/2018/04/10/oauth-authorization-code-grant-type](https://developer.okta.com/blog/2018/04/10/oauth-authorization-code-grant-type)

OAUTH 2.0<br/>
[https://oauth.net/2/](https://oauth.net/2/)

OAUTH CSRF 대응방안<br/>[https://grooveshark.tistory.com/209](https://grooveshark.tistory.com/209)

KAKAO DEVELOPERS - ID TOKENS<br/>[https://developers.kakao.com/docs/latest/ko/kakaologin/common#oidc-id-token](https://developers.kakao.com/docs/latest/ko/kakaologin/common#oidc-id-token)

JWT 디코더 웹사이트<br/>[https://jwt.io/](https://jwt.io/)