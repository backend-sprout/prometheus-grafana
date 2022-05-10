# HTTPS and authentication

프로메테우스는 기본/TLS 인증을 지원한다.(다만 아직 실험적 기능이다.)   

로드할 웹 설정 파일을 지정할 땐 `--web.config.file` 플래그를 사용한다.   
로드한 파일은 YAML 형식으로 작성하며, 아래와 같은 스키마로 정의한다.   

설정 변경과 같은 http 요청이 들어올 때마다 파일을 읽어들이며, 즉시 인증서들을 가져온다.   

**공통 플레이스 홀더**   
* `<boolean>` : true of false      
* `<filename>` : 현재 작업 디렉토리 안에 있는 유효한 경로   
* `<secret>` : 비밀번호 같이 평소 시크릿에 사용하는 문자열  
* `<string>` : 평범한 문자열 


```yml
tls_server_config:
  # 서버가 클라이언트에 인증할 때 사용할 인증서 파일과 키 파일.
  cert_file: <filename>
  key_file: <filename>

  # 클라이언트 인증을 위한 서버 정책. ClientAuth 정책에 매핑된다.
  # clientAuth 옵션에 관한 자세한 내용은 아래 사이트를 참고해라:
  # https://golang.org/pkg/crypto/tls/#ClientAuthType
  #
  # 주의: 클라이언트 인증을 활성화하고 싶다면
  # RequireAndVerifyClientCert를 사용해야 한다. 다른 정책은 안전하다고 볼 수 없다.
  [ client_auth_type: <string> | default = "NoClientCert" ]

  # 서버로 전송한 클라이언트 인증서를 인증하기 위한 CA 인증서.
  [ client_ca_file: <filename> ]

  # 허용할 TLS 최소 버전.
  [ min_version: <string> | default = "TLS12" ]

  # 허용할 TLS 최대 버전.
  [ max_version: <string> | default = "TLS13" ]

  # TLS 1.2 버전까지를 위한 지원하는 암호화 스위트(cipher suite) 목록.
  # 비어 있으면 Go 디폴트 암호화 스위트를 사용한다.
  # 사용 가능한 암호화 스위트들은 go 문서에서 설명하고 있다:
  # https://golang.org/pkg/crypto/tls/#pkg-constants
  [ cipher_suites:
    [ - <string> ] ]

  # prefer_server_cipher_suites에는 서버가 클라이언트에서 가장 선호하는 암호화 스위트를 선택할지,
  # 서버에서 가장 선호하는 암호화 스위트를 선택할지를 지정한다.
  # true면 서버의 선호도대로 cipher_suites에 지정한 순서를 따른다.
  [ prefer_server_cipher_suites: <bool> | default = true ]

  # ECDHE 핸드셰이크에서 사용할 elliptic curve를 선호하는 순서대로 지정한다.
  # 사용 가능한 curve들은 go 문서에서 설명하고 있다:
  # https://golang.org/pkg/crypto/tls/#CurveID
  [ curve_preferences:
    [ - <string> ] ]

http_server_config:
  # HTTP/2 지원을 활성화한다. 참고로 HTTP/2는 TLS를 함께 사용할 때만 지원한다.
  # 이 설정은 실행 중일 때 즉석으로 변경할 수 없다.
  [ http2: <boolean> | default = true ]
  # HTTP 응답에 추가할 수 있는 헤더 목록.
  [ headers:
    # HTTP 응답에 Content-Security-Policy 헤더를 설정한다.
    # 비어있으면 설정하지 않는다.
    [ Content-Security-Policy: <string> ]
    # HTTP 응답에 X-Frame-Options 헤더를 설정한다.
    # 비어있으면 설정하지 않는다. deny와 sameorigin을 사용할 수 있다.
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
    [ X-Frame-Options: <string> ]
    # HTTP 응답에 X-Content-Type-Options 헤더를 설정한다.
    # 비어있으면 설정하지 않는다. nosniff를 사용할 수 있다.
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
    [ X-Content-Type-Options: <string> ]
    # HTTP 응답에 X-XSS-Protection 헤더를 설정한다.
    # 비어있으면 설정하지 않는다. nosniff를 사용할 수 있다.
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection
    [ X-XSS-Protection: <string> ]
    # HTTP 응답에 Strict-Transport-Security 헤더를 설정한다.
    # 비어있으면 설정하지 않는다.
    # 이 헤더를 사용하면 브라우저에서 동일한 도메인과 서브 도메인에서 호스팅하는
    # 프로메테우스와 다른 애플리케이션들을 강제로 HTTPS로 로드할 수 있으므로 주의해서 사용해야 한다.
    # https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security
    [ Strict-Transport-Security: <string> ] ]

# 기본 인증을 통해 웹 서버에 대한 모든 액세스 권한을 부여하는 username과 해싱한 password 목록.
# 비어 있으면 기본 인증이 요구하지 않는다. password는 bcrypt로 해싱한다.
basic_auth_users:
  [ <string>: <secret> ... ]
```

유효한 [예제 파일 링크](https://github.com/prometheus/prometheus/blob/release-2.32/documentation/examples/web-config.yml)     

