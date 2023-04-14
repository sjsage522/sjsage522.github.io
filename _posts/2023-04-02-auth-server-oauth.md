---
title: (SpringBoot) [Oauth2.0, JWT] 소셜로그인 기능 구현하기(인증서버 구축하기)
author: jun
date: 2023-04-03 00:37:00 +0900
categories: [SpringBoot, Feature]
tags: [jwt, oauth, springboot]
render_with_liquid: false
comments: true
img_path: /assets/img/post/20230402/
---

<span style="color:grey">본 포스트는 SpringBoot를 활용하여 구글,카카오 로그인과 같은 소셜로그인 기능을 구현하는 과정을 기술합니다.</span>

Oauth2.0 프로토콜 스펙에 따라 구현 및 JWT 인증방식을 사용합니다.

프로젝트에서 **Dependency**는 다음과 같습니다.

> [SpringBoot 2.7.8]

```yml
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    
    implementation 'io.jsonwebtoken:jjwt:0.9.1'
    implementation 'commons-validator:commons-validator:1.7'
    implementation 'com.googlecode.json-simple:json-simple:1.1.1'

    compileOnly 'org.projectlombok:lombok'
    
    runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
    runtimeOnly 'com.h2database:h2'
    
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## Oauth2.0 프로토콜

[Oauth2.0 Authorization Code Grant](/posts/oauth-authorization-code/) 방식으로 구현하였습니다. 

## JWT (Json Web Token)

## 구글 서비스 등록하기

[구글 클라우드 플랫폼](https://console.cloud.google.com/) 에서 프로젝트 생성 후, **클라이언트 ID**(client-id)와 **클라이언트 보안 비밀번호**(secret-key)를 발급받습니다.

**새 프로젝트 생성** > 프로젝트 이름을 정하고 만들기 버튼을 클릭합니다.

![image-20230402175416878](image-20230402175416878.png)
_프로젝트를 생성하기_

<br/>

생성한 프로젝트에 진입 후, **사용자 인증 정보 만들기** 버튼을 클릭 > **OAuth 클라이언트 ID** 버튼을 클릭합니다.

![image-20230402180643333](/image-20230402180643333.png)
_사용자 인증정보 만들기_

<br/>

OAuth 클라이언트 ID를 만들기위해 먼저 동의화면을 구성하라는 안내가 표시됩니다. User Type은 외부로 체크 후, **동의 화면**을 구성해 줍니다.

<u>앱 이름, 사용자 지원 이메일, 개발자 연락처 정보</u>는 **필수 사항**입니다. 앱 로고와 앱 도메인 정보들은 사용자가 소셜 로그인 시에, 표시되는 화면 정보입니다.

필요한 정보들을 기입한 후에, **저장 후 계속** 버튼을 클릭합니다.

![image-20230402181413803](/image-20230402181413803.png)

![image-20230402181525194](/image-20230402181525194.png)
_동희 화면 구성하기_

<br/>

**범위 추가 또는 삭제** 버튼을 클릭하여, 사용자 데이터 엑세스 범위를 추가합니다(앱에서 접근할 수 있는 사용자 데이터 범위를 추가)

<u>email, profile, openid</u> 범위를 추가한 후, **저장 후 계속** 버튼을 클릭합니다.

![image-20230402182419936](/image-20230402182419936.png)
_엑세스 범위 추가하기_

<br/>

특정 테스트 계정을 추가하고 싶다면, **ADD USERS** 버튼을 클릭하여 테스트 유저를 추가해줍니다. (option)

저장 후 계속 > 요약 정보를 확인 후, **대시보드로 돌아가기** 버튼을 클릭합니다.

![image-20230402182652179](/image-20230402182652179.png)
_테스트 유저 추가하기_

<br/>

이제, OAuth 클라이언트 ID를 만들 수 있습니다. <u>애플리케이션 유형, 이름, 승인된 리디렉션 URI</u>를 등록합니다.

> 승인된 리디렉션 URI란?
>
> 승인된 리디렉션 URI(callback uri)는 인증 서버(google, kakao ...)에서 사용자를 인증한 후, 클라이언트 앱에서 인가 코드를 받기 위해 리디렉션하는 uri입니다. 
> {: .prompt-info }

![image-20230402183235184](/image-20230402183235184.png)
_OAuth 클라이언트 ID 만들기_

<br/>

다음과 같이, 구글에 대한 <u>클라이언트 ID와 보안 비밀번호</u>가 생성됩니다.

![image-20230402191512646](/image-20230402191512646.png){: w="300" }
_OAuth 클라이언트 생성_

<br/>

## 카카오 서비스 등록하기

[KaKao Developers](https://developers.kakao.com/)로 이동합니다.

내 애플리케이션 > 애플리케이션 추가하기 > <u>앱 이름, 사업자명</u> 입력 후 **저장** 버튼을 클릭합니다.

![image-20230402192058997](/image-20230402192058997.png)
_애플리케이션 추가하기_

<br/>

<u>REST API키(클라이언트 ID)</u>를 확인할 수 있습니다.

![image-20230402192807463](/image-20230402192807463.png)
_클라이언트 ID 확인하기_

<br/>

제품 설정 > 카카오 로그인에서 **활성화 설정 ON** 이후 나타나는 **OpenID Connect 활성화 설정도 ON**으로 설정합니다.

**Redirect URI도 등록**합니다.

![image-20230402193316185](/image-20230402193316185.png)
_활성화 및 callback 등록_

<br/>

내 애플리케이션 > 앱 설정(프로젝트 클릭)에서 왼쪽 상단의 **☰** 클릭 후, **동의항목** 버튼을 클릭합니다.

각 항목마다 설정 버튼을 클릭하여 **닉네임, 프로필 사진, 카카오계정(이메일)에 대한 동의 항목을 설정**합니다.

![image-20230402194136533](/image-20230402194136533.png)
_동의항목 설정하기_

<br/>

**☰**(메뉴)에서 **보안**버튼을 클릭 후, **Client Secret 코드를 생성**합니다. 활성화 상태를 '<u>사용함</u>'으로 변경합니다.

![image-20230402194502627](/image-20230402194502627.png){: w="300" } 

<br/>

## 소셜 로그인 기능구현 플로우

> **프로젝트 전체 소스코드는 [이곳](https://github.com/sjsage522/member)에서 확인할 수 있습니다.**

### OAuth2.0 동작과정

![image-oauth2.0-process.png](/oauth2.0-process.png)
_Oauth2.0 동작과정_

### Resource Owner의 로그인 요청

Resource Owner(사용자)는 구글, 카카오 등과 같은 **소셜 서비스에 대한 로그인 요청**을 하게됩니다. 해당 요청을 처리하는 컨트롤러(OAuth2LoginController)를 구현합니다.

```java
package com.junseok.member.oauth;

import com.junseok.member.common.property.JwtProperty;
import com.junseok.member.common.property.OAuthProperty;
import com.junseok.member.common.validation.annotation.UrlValid;
import com.junseok.member.util.CookieUtils;
import com.junseok.member.util.SessionScopeUtils;

import lombok.RequiredArgsConstructor;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletResponse;
import javax.validation.Valid;
import java.io.IOException;

@Validated
@RequiredArgsConstructor
@RequestMapping("/oauth2/authorize")
@RestController
public class OAuth2LoginController {
    private final OAuth2LoginService oauth2LoginService;
    private final CookieUtils cookieUtils;
    private final SessionScopeUtils sessionScopeUtils;
    private final OAuthProperty oauthProperty;
    private final JwtProperty jwtProperty;

    /**
     * @param redirectUri 클라이언트 redirect uri
     * @return resource server authorization 301 redirect
     * @throws IOException
     */
    @GetMapping("/{service_provider}")
    public void authorizeService(@PathVariable(value = "service_provider") String serviceProvider, @RequestParam(value="redirect_uri") @Valid @UrlValid String redirectUri, HttpServletResponse response) throws IOException {
        String authorizeUri = oauth2LoginService.getAuthorizeUri(serviceProvider);

        cookieUtils.addCookie("redirect_uri", redirectUri, oauthProperty.getCookiePath(), oauthProperty.getCookieMaxAge());
        cookieUtils.addCookie("service_provider", serviceProvider, oauthProperty.getCookiePath(), oauthProperty.getCookieMaxAge());

        response.sendRedirect(authorizeUri);
    }

    /**
     * service provider별 인증 후 callback
     * @param redirectUri       access token 발급 후 client redirect uri
     * @param serviceProvider   service provider (google, local ... etc)
     * @param authorizationCode /oauth2/authorize/{provider} 요청 후 발급 code
     * @param state             CSRF 방어 state value
     * @throws IOException
     */
    @GetMapping("/login/callback")
    public void loginCallback(
            @CookieValue(value = "redirect_uri") String redirectUri,
            @CookieValue(value = "service_provider") String serviceProvider,
            @RequestParam(value = "code") String authorizationCode,
            @RequestParam(value = "state") String state,
            HttpServletResponse response) throws IOException {

        if (!state.equals((String)sessionScopeUtils.getAttribute("state"))) {
            throw new IllegalArgumentException("Failed to authentication. Cause: Mismatch state value.");
        }
        
        String providerAccessToken = oauth2LoginService.getAccessToken(serviceProvider, authorizationCode);

        String jwt = oauth2LoginService.createJwt(providerAccessToken, serviceProvider);
        String refreshToken = oauth2LoginService.createRefreshToken();

        cookieUtils.addCookie("auth_token", jwt, "/", jwtProperty.getCookieMaxAge());
        cookieUtils.addCookie("auth_refresh", refreshToken, "/", jwtProperty.getCookieMaxAge());
        
        response.sendRedirect(redirectUri);
    }
}
```

### Client(Application)의 로그인 요청과 로그인 페이지 제공

#### authorizeService 메소드 요청

1. 사용자는 소셜 로그인을 위해 클라이언트 앱으로 다음과 같은 리소스로 HTTP 요청을 합니다. <br/>
   <u>GET /oauth2/authorize/kakao?redirect_uri=https://example.com</u>

2. oauth2LoginService.getAuthorizeUri 메소드로 부터 리디렉션할 uri를 얻고, 브라우저 쿠키에 <u>redirect_uri, service_provider</u> 정보를 저장합니다. <br/>

   > **redirect_uri란?** <br/>
   > OAuth 동작 과정중 AuthorizationServer로 부터 AccessToken을 발급 받고 DB에 저장한 후(로그인 성공처리), 사용자를 리디렉션 시켜줄 uri입니다.

   > **service_provider란?** <br/>
   > google, kakao 와 같은 리소스 제공자를 의미합니다. AuthorizationCode 또는 AccessToken을 얻을 때 해당 값을 사용합니다.

   > service layer에서는 csrf 공격 방지를 위한 state값을 세션에 저장합니다.
   >
   > 이러한 처리는 필수사항은 아니지만, 보안 취약점 공격을 방어하기 위해서 구현을 강력히 권장합니다. 

3. 사용자를 소셜 로그인 화면으로 리디렉션 시킵니다.

### ID/PW 제공 및 Authorization Code 발급

![image-20230402223750140](/image-20230402223750140.png){: w="400" }
_카카오 로그인 화면_

### RedirectUri(CallbackUri) 로 리다이렉트 및 JWT 발급

#### loginCallback 메소드 요청

1. 사용자가 아이디(이메일), 패스워드를 통해 인증 후, 정보 제공에 대한 동의까지 완료했다면, <br/>인증 서버에서 클라이언트 앱의 <u>GET /oauth2/authorize/login/callback 으로 요청</u>하게 됩니다.

   > 각 소셜 서비스에 등록한 RedirectUri(CallbackUri)로 요청하게 됩니다.
   >
   > 이 때, 브라우저(사용자)에서는 인증 서버에서 제공한 AuthorizationCode와 state값을 쿼리파라미터로 전달합니다.

2. oauth2LoginService.getAccessToken 메소드에서 인증 서버로 부터 AccessToken(JWT)를 얻고, 클라이언트 앱에서 사용하기 위한 JWT와 RefreshToken을 생성합니다.

   > JWT와 RefreshToken는 클라이언트의 쿠키에 저장하도록 합니다. <br/>
   > 해당 값들을 클라이언트에 저장하는 방법은 여러가지가 있지만, 필자는 HTTP only 속성의 쿠키로 저장하는 것이 보안상 안전하다고 생각합니다.
   >
   > 단, 이렇게 저장하게 되면 CSRF(*Cross Site Request Forgery*)공격을 방지하기 위한 보안코딩이 필수로 요구됩니다. <br/>일회성 토큰(csrf token)을 이용하거나, 레퍼러 체크 등의 방법으로 보안코드를 구현하도록 합니다. <br/>
   > 또한, 자바스크립트 코드를 이용해 민감한 정보가 탈취당하지 않도록 XSS(*Cross Site Scripting*)에 대한 보안코드도 구현하도록 합니다.

3. 쿠키로 전달된 redirect_uri로 사용자를 리디렉션 시켜주면서, 로그인 과정을 완료합니다.

   1. 이제부터 인증이 필요한 리소스 요청에 대해서, 쿠키에 저장되어 있는 토큰을 이용해 인증과정을 거치게 됩니다.

<br/>

큰 흐름은 이렇게 구성됩니다. 이제 프로젝트의 service layer를 살펴보겠습니다.

<br/>

## 프로젝트 상세 구현

service layer에서 어떤식으로 사용자에게 로그인 페이지를 제공하고, 인증 코드를 이용해 AccessToken을 얻는지 확인 해봅니다.

전체 코드는 다음과 같습니다.

```java
package com.junseok.member.oauth;

import java.util.*;

import com.junseok.member.common.exception.ErrorCode;
import com.junseok.member.common.exception.NotFoundEmailException;
import com.junseok.member.common.property.OAuthProviderProperty;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.json.simple.parser.ParseException;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponents;
import org.springframework.web.util.UriComponentsBuilder;

import com.junseok.member.user.AuthProvider;
import com.junseok.member.user.User;
import com.junseok.member.user.UserRepository;
import com.junseok.member.util.CommonUtils;
import com.junseok.member.util.JwtProvider;
import com.junseok.member.util.SessionScopeUtils;

import lombok.RequiredArgsConstructor;

import org.springframework.stereotype.Service;

import javax.transaction.Transactional;

@RequiredArgsConstructor
@Service
public class OAuth2LoginService {
    private final OAuthPropertyFactory oAuthPropertyFactory;
    private final RestTemplate restTemplate = new RestTemplate();

    private final SessionScopeUtils sessionScopeUtils;
    private final CommonUtils commonUtils;
    private final JwtProvider jwtProvider;
    private final UserRepository userRepository;
    private final RefreshTokenRepository refreshTokenRepository;

    /**
     * Url about AuthorizationCode
     * @param service ServiceProvider
     */
    public String getAuthorizeUri(String service) {
        final OAuthProviderProperty oAuthProviderProperty = oAuthPropertyFactory.getOAuthProviderProperty(service)
                .orElseThrow(() -> new IllegalArgumentException("Oauth2 Service Provider is not Valid."));

        String state = commonUtils.generateStateValue(40);
        sessionScopeUtils.setAttribute("state", state);

        UriComponentsBuilder uriComponentsBuilder = UriComponentsBuilder.newInstance();
        switch (service) {
            case "google":
                uriComponentsBuilder
                        .scheme("https")
                        .host("accounts.google.com")
                        .path("/o/oauth2/v2/auth");
            break;
            case "kakao":
                uriComponentsBuilder
                        .scheme("https")
                        .host("kauth.kakao.com")
                        .path("/oauth/authorize");
            break;
        }

        UriComponents uriComponents = uriComponentsBuilder
                .queryParam("client_id", oAuthProviderProperty.getClientId())
                .queryParam("redirect_uri", oAuthProviderProperty.getRedirectUri())
                .queryParam("response_type", oAuthProviderProperty.getResponseType())
                .queryParam("scope", oAuthProviderProperty.getScope())
                .queryParam("state", state)
                .build(true);

        return uriComponents.toString();
    }

    /**
     * Get AccessToken (JWT) from ServiceProvider
     * @param service ServiceProvider
     * @param authorizationCode AuthorizationCode
     */
    public String getAccessToken(String service, String authorizationCode) {
        final OAuthProviderProperty oAuthProviderProperty = oAuthPropertyFactory.getOAuthProviderProperty(service)
                .orElseThrow(() -> new IllegalArgumentException("Oauth2 Service Provider is not Valid."));

        UriComponentsBuilder uriComponentsBuilder = UriComponentsBuilder.newInstance();
        switch (service) {
            case "google":
                uriComponentsBuilder = UriComponentsBuilder.newInstance()
                        .scheme("https")
                        .host("oauth2.googleapis.com")
                        .path("/token");
                break;
            case "kakao":
                uriComponentsBuilder = UriComponentsBuilder.newInstance()
                        .scheme("https")
                        .host("kauth.kakao.com")
                        .path("/oauth/token");
                break;
        }
        String accessTokenEndpoint = uriComponentsBuilder.build(true).toString();

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.setAccept(List.of(MediaType.APPLICATION_JSON));

        MultiValueMap<String, String> bodyData = new LinkedMultiValueMap<>();
        bodyData.add("client_id", oAuthProviderProperty.getClientId());
        bodyData.add("client_secret", oAuthProviderProperty.getSecretId());
        bodyData.add("code", authorizationCode);
        bodyData.add("grant_type", "authorization_code");
        bodyData.add("redirect_uri", oAuthProviderProperty.getRedirectUri());

        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(bodyData, headers);
        ResponseEntity<String> response = restTemplate.postForEntity(accessTokenEndpoint, request, String.class);

        JSONParser jsonParser = new JSONParser();
        try {
            JSONObject jsonObject = (JSONObject) jsonParser.parse(response.getBody());
            return (String) jsonObject.get("id_token");
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * Get JWT for App Service
     */
    @Transactional
    public String createJwt(String providerAccessToken, String serviceProvider) {
        Map<String, Object> payloads = jwtProvider.getPayload(providerAccessToken);

        String email = (String) payloads.get("email");
        if (email == null || email.isEmpty()) {
            throw new NotFoundEmailException(ErrorCode.NOT_FOUND_EMAIL);
        }

        String picture = (String) payloads.get("picture");
        String name = (String) payloads.get("name");
        String username = name != null ? name : (String) payloads.get("nickname");
        AuthProvider authProvider = AuthProvider.valueOf(serviceProvider.toUpperCase());

        User findUser = userRepository.findByEmail(email).orElseGet(() -> {
            User newUser = User.builder()
                .email(email)
                .picture(picture)
                .username(username)
                .authProvider(authProvider)
                .build();
            userRepository.save(newUser);
            return newUser;
        });
        findUser.updateUser(picture, username, authProvider);

        return jwtProvider.createToken(String.valueOf(findUser.getId()), findUser.getEmail());
    }

    /**
     * Get RefreshToken
     */
    @Transactional
    public String createRefreshToken() {
        RefreshToken refreshToken = jwtProvider.createRefreshToken();
        refreshTokenRepository.save(refreshToken);

        return refreshToken.getRefreshToken();
    }
}
```

### getAuthorizeUri 메소드

소셜 로그인 화면으로 리디렉션 하기위한 uri를 얻어오는 메소드입니다. 각 서비스별로 엔드포인트가 다르기 때문에, 파라미터로 받은 service로 분기처리하여 uri를 만들어줍니다.

이곳에서 uri를 만들 때, 각 서비스에서 발급받은 <u>클라이언트 ID, RedirectUri(CallbackUri), scope(email, username 등과 같은 사용자 정보범위), state</u> 값들을 쿼리파라미터로 함께 전달합니다.

Oauth와 관련된 변수들은 관리에 용이하도록 프로퍼티로 관리해줍니다.

> 각 서비에 등록된 redirectUri와 쿼리파라미터의 redirectUri는 동일해야 합니다.
>
> 악의적인 누군가들은 redirectUri를 유추할 수도 있습니다. 때문에, 보안강화(CSRF 공격 방지)를 위해서 state값을 추가로 전달해줍니다. 이 state값은 인증서버에서 redirectUri로 리디렉션할때 쿼리파라미터로 함께 전달해줍니다. <br/>따라서, 클라이언트 앱에서 검증로직을 직접구현해야 합니다.

### getAccessToken 메소드

소셜 로그인 인증을 완료했다면, 인증서버는 클라이언트 앱의 RedirectUri(CallbackUri)로 리디렉션 합니다. 이 때, 브라우저에서 AuthorizationCode를 함께 전송합니다. (인증코드 요청 시 state값을 전달했다면 state값도 전송)

해당 메소드에서는 인증코드로 각 서비스의 **인증토큰을 요청**하는 역할을 수행합니다.

해당 요청은 POST 요청입니다. 따라서, body정보에 발급받은 <u>클라이언트 ID, 클라이언 비밀 키(Secret Key)와 인증코드, grant_type("authorization_code" 고정)</u>을 담아 요청합니다.

응답 데이타를 JSON 형식으로 변환 후, id_token값을 얻어 리턴합니다. 이 값이 각 서비스에서 제공한 JWT 입니다.

### createJwt 메소드

클라이언트에서 사용할 **JWT를 발급**하는 메소드 입니다. 최종적으로 해당 토큰을 통해 클라이언트 앱에 대한 인증처리를 수행합니다.

각 서비스에서 발급받은 JWT를 파싱하여 페이로드로 부터, scope에 대한 값(email, picture, name 등)들을 얻어줍니다.

유니크한 값인 email로 디비로 부터 사용자를 조회합니다. 만약 디비에 값이 없다면 최초로 가입한 사용자이므로, 디비에 INSERT 해줍니다.

jwtProvider에서 JWT를 만들고 리턴합니다.

### JwtProvider

해당 클래스에서는 JWT, RefreshToken을 발급하고, JWT에 대한 유효성 검사를 수행합니다. [이곳](https://github.com/sjsage522/member/blob/main/src/main/java/com/junseok/member/util/JwtProvider.java)에서 소스코드를 확인할 수 있습니다.

### Propety

JWT, Oauth의 관한 변수들을 프로퍼티로 관리합니다. [이곳](https://github.com/sjsage522/member/tree/main/src/main/resources)에서 YML 정보들을 확인할 수 있습니다.

### Utils

공통적으로 사용할 수 있는 메소드들이 정의된 유틸리티 클래스는 [이곳](https://github.com/sjsage522/member/tree/main/src/main/java/com/junseok/member/util)에서 확인할 수 있습니다.

<br/>

## 인증 (Authentication)

[SSOController](https://github.com/sjsage522/member/blob/main/src/main/java/com/junseok/member/sso/SSOController.java) 클래스에서 인증 API를 구현합니다.<br/>사용자가 요청한 AccessToken(JWT)과 RefreshToken에 대한 유효성을 검증합니다.

[SSOService](https://github.com/sjsage522/member/blob/main/src/main/java/com/junseok/member/sso/SSOService.java) 클래스에서 구체적인 검증로직을 작성합니다. 코드는 다음과 같습니다.

```java
package com.junseok.member.sso;

import java.util.Map;

import javax.transaction.Transactional;

import com.junseok.member.util.RedisUtils;
import org.springframework.stereotype.Service;

import com.junseok.member.common.exception.ErrorCode;
import com.junseok.member.common.exception.ExpiredRefreshTokenException;
import com.junseok.member.common.exception.NotFoundRefreshTokenException;
import com.junseok.member.common.property.JwtProperty;
import com.junseok.member.oauth.RefreshToken;
import com.junseok.member.oauth.RefreshTokenRepository;
import com.junseok.member.util.CookieUtils;
import com.junseok.member.util.JwtProvider;

import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
@Service
public class SSOService {
    private final JwtProvider jwtProvider;
    private final RefreshTokenRepository refreshTokenRepository;

    private final CookieUtils cookieUtils;

    private final JwtProperty jwtProperty;

    private final RedisUtils redisUtil;

    @Transactional(dontRollbackOn = ExpiredRefreshTokenException.class)
    public Boolean authentication(String accessToken, String refreshToken) {
        if (jwtProvider.validateToken(accessToken)) {
            return Boolean.TRUE;
        }

        // Expired JWT -> Refresh Access Token
        RefreshToken findRefreshToken = refreshTokenRepository.findByRefreshToken(refreshToken)
            .orElseThrow(() -> new NotFoundRefreshTokenException(ErrorCode.NOT_FOUND_REFRESH_TOKEN));
        
        // Expired RefreshToken -> throw Error
        if (findRefreshToken.isExpired()) {
            refreshTokenRepository.delete(findRefreshToken);
            throw new ExpiredRefreshTokenException(ErrorCode.EXPIRED_REFRESH_TOKEN);
        }

        Map<String, Object> payloads = jwtProvider.getPayload(accessToken);
        String subject = (String) payloads.get("sub");
        String email = (String) payloads.get("email");

        // JWT, Refresh Token 재발급
        String newAccessToken = jwtProvider.createToken(subject, email);
        RefreshToken newRefreshToken = jwtProvider.createRefreshToken();

        findRefreshToken.update(newRefreshToken.getRefreshToken(), newRefreshToken.getExpiresTime());

        cookieUtils.addCookie("auth_token", newAccessToken, "/", jwtProperty.getCookieMaxAge());
        cookieUtils.addCookie("auth_refresh", newRefreshToken.getRefreshToken(), "/", jwtProperty.getCookieMaxAge());

        return Boolean.TRUE;
    }

    @Transactional
    public Boolean revokeToken(String accessToken, String refreshToken) {
        if (!jwtProvider.validateToken(accessToken)) {
            return Boolean.FALSE; // Expired JWT
        }

        // delete refreshToken
        RefreshToken targetRefreshToken = refreshTokenRepository.findByRefreshToken(refreshToken)
            .orElseThrow(() -> new NotFoundRefreshTokenException(ErrorCode.NOT_FOUND_REFRESH_TOKEN));
        refreshTokenRepository.delete(targetRefreshToken);

        // Set BlackList
        Long expiredTime = (Long) jwtProvider.getPayload(accessToken).get("exp");
        redisUtil.setBlackList(accessToken, "access_token", expiredTime);

        return Boolean.TRUE;
    }
}
```

### authentication 메소드

먼저, JWT가 유효한지 검증합니다. 만약 JWT가 유효하지 않다면 에러를 발생시킵니다.

JWT가 만료되었다면, 리프레시토큰으로 재발급을 시도합니다. 만약, 디비에 리프레시토큰이 존재하지 않거나 만료되었다면, 에러를 발생시킵니다.

리프레시토큰이 존재한다면, JWT와 리프레시 토큰을 새로 발급합니다.

### revokeToken 메소드

로그아웃 기능입니다. **블랙리스트 방식**을 통해 사용자의 JWT를 레디스 저장소에 블랙리스트로서 등록하고, 리프레시토큰을 디비에서 제거합니다.

로그아웃 기능 호출 후 인증 메소드를 호출하면, 리프레시 토큰을 찾을 수 없기 때문에 사용자는 다시 로그인해야 합니다.

<br/>

## 마침

> 궁금하시거나, 부족한 내용이 있다면 코멘트 부탁드립니다.  🙇🏻‍♂️

<br/>

## 레퍼런스

[https://hudi.blog/oauth-2.0/](https://hudi.blog/oauth-2.0/)



카카오 로그인에 대한 전반적인 내용이 담긴 문서입니다.<br/>[https://developers.kakao.com/docs/latest/ko/kakaologin/common](https://developers.kakao.com/docs/latest/ko/kakaologin/common)



구글 Oauth2.0 엔드포인트에 대한 문서입니다.<br/>[https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow?hl=ko#oauth-2.0-endpoints](https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow?hl=ko#oauth-2.0-endpoints)
