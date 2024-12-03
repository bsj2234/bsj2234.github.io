---
layout: post
title: "AWS Cognito Unity"
date: 2024-12-03
categories: [Network]
tags: [AWS, Cognito, Unity]
media_subpath: /medias/posts
---

# AWS Cognito Unity
[참고 영상](https://www.youtube.com/watch?v=lzQ2rLqlqyk)
 - 구글 로그인을 지원하고 싶었다
 - 하지만 이 영상은 IOS기기만 지원한다
 - Deeplink제공을 ios에서 제공한 파일을 서버에 넣어둬야함
 - 서버도 필요함


## 힘들었던 점
  - Domain을 내가 구매한 도메인을 쓰려고 하는데 잘 안됐다
  - 일단 그래서 기본 도메인으로 구현중
  - 인증서, 도메인구매, 레코드 설정 등 할게 너무 많음
## 구현
  - aws cognito 설정

## 작동 방식
 - cognito 에 인증 요청
 - 구글  로그인하면 redirect로 이동하며 code를 받아redirect 뒤에 붙여서 반환해줌
 - 그럼 유니티에서 웹서버 리슨하며 코드를 받음
 - 코드를 받아서 유니티에 저장 그리고 인증여부를 보낼때 사용 하는듯



## Cognito user pool 생성
 - 어플리케이션 타입은 모바일 앱
 - 로그인 Identifiers는 email
 - Required attributes는 이메일
 - Return url은 callback URL인듯 나중에도 설정 가능
## App client information -> Edit
 - App client setting 에서 authentication flows 모두 체크해제(Get new user tokens from existing authenticated sessions: ALLOW_REFRESH_TOKEN_AUTH 는 그대로 유지ㅣ)
 - 나머지는 기본 Enable token revocation는 켜져있게 냅둠
## Sidebar -> Social and external providers
  - Google 추가
    - google console 에서 client id, client secret를 복사해 추가 (이것도 처음하면 뭐 많이 해야한다)
      - *google 콘솔에서 해야하는 일* 
      - google 클라 설정 전에 앱 설정에 authoized domains 에 amazoncognito.com 추가 커스텀 도메인도 쓴다면 추가
      - Javascript origins에는 aws userpool->branding->domain 주소 넣고
      - Allowed redirect URLs에는 ***위의 cognito주소/oauth2/idpresponse 를 넣어줘야함***
    - Map 은 이메일에 이메일 해주면되는듯 sub도 필요하면 추가 영상에서는 sub로 Username 추가
    - Scope 설정 profile email openid 설정(영상에서는 구글 콘솔에서했는데 바뀐듯?)
    - Map email To email, username to sub 으로 설정해봄  (영상과 좀 다름)
## Login page -> Edit
  - Callback urls
    - 이건 클라에서 지정되는 callbackurl을 허용해줌(즉 callback url은 클라에서 지정됨)
  - Allowed sign-out urls
    - 로그아웃 후 리다이렉트 되는 url을 허용해줌(아마?)
  - Identity providers
    - 기본 Cognito User Pool
    - Google 추가
    - OpenId Connect scopes 이것도 아마 Email OpenId Profile 로 해야하지 않을까?
## 람다 함수 설정
  - 아무 함수나 테스트용으로 만듬
## GatewayAPI 설정
  - Authorizers -> Create new authorizer
    - Cognito User Pool 선택
    - TokenSource -> Authorization
    - DeployAPI -> new stage 생성 -> Deploy
  - REST GET으로 람다 메서드 연결
  - Get -> Authorization -> 위에서 만든 권한 선택
  - 사실 이거 왜하는지 잘 모르겠음
## Unity
  - [참고 링크](https://github.com/BatteryAcid/unity-cognito-hostedui-social-client)
  - 여기에는 IOS 딥링크를 사용하는거밖에 없음
  - 그래서 추가 구현
```csharp
// 추가된 일부 코드
//에디터의 경우 웹서버를 위해 추가
#if UNITY_EDITOR || UNITY_STANDALONE_WIN
using System.Net;
#endif

public class AuthenticationManager : MonoBehaviour
{
   public static string CachePath;

   // In production, should probably keep these in a config file
   private const string AppClientID = "****************"; // App client ID, found under App Client Settings
   //기존 코드는 Prefix와 도메인 주소 둘 다 있음
   //일단 이것만 사용하는걸로 함
   private const string AuthCognitoDomainPrefix = "************"; // Found under App Integration -> Domain Name. Changing this means it must be updated in all linked Social providers redirect and javascript origins

#if UNITY_EDITOR || UNITY_STANDALONE_WIN
    // 에디터의 경우 웹서버를 위해 추가
    // cognito의 allowed redirect urls에 추가된 주소를 넣어야함
        private const string RedirectUrl = "http://localhost:8080/callback";
        private HttpListener _httpListener;
        //콜백 리슨
        public async void StartLocalServer()
        {
            _httpListener = new HttpListener();
            _httpListener.Prefixes.Add("http://localhost:8080/");
            _httpListener.Start();
            
            while (true)
            {
                var context = await _httpListener.GetContextAsync();
                var code = context.Request.QueryString["code"];
                if (!string.IsNullOrEmpty(code))
                {
                    await ExchangeAuthCodeForAccessToken($"http://localhost:8080/callback?code={code}");
                    // 브라우저에 성공 메시지 표시
                    var response = context.Response;
                    string responseString = "<html><body>인증 성공! 이 창을 닫아도 됩니다.</body></html>";
                    byte[] buffer = System.Text.Encoding.UTF8.GetBytes(responseString);
                    response.ContentLength64 = buffer.Length;
                    response.OutputStream.Write(buffer, 0, buffer.Length);
                    response.OutputStream.Close();
                }
            }
        }


/*
Success, Code exchange complete!
UnityEngine.Debug:Log (object)
AuthenticationManager/<CallCodeExchangeEndpoint>d__13:MoveNext () (at Assets/00_ProjectCCC/00_Scripts/04_Auth/AuthenticationManager.cs:130)
System.Threading.Tasks.TaskCompletionSource`1<object>:SetResult (object)
ExtensionMethods/<>c__DisplayClass0_0:<GetAwaiter>b__0 (UnityEngine.AsyncOperation) (at Assets/00_ProjectCCC/00_Scripts/04_Auth/Utilities/ExtensionMethods.cs:13)
UnityEngine.AsyncOperation:InvokeCompletionEvent ()
*/
```


## IOS의 경우
  - 딥링크 설정
  - 애플에서 판매자 등록 후 받을 수 있는 딥링크를 ec2에 추가하면 웹페이지를 통해 app 실행 가능
## 자격증명과 유저풀 차이
 - 자격증명은 임시 권한임 지금은 해당안됨
 - 유저풀은 인증 후 유저정보를 저장
## url 틀려서 forbidden 오류 발생
 - 웹서버 주소 확인
## 토큰
 - 토큰을 받아오는데 api 실행시 오류발생
 - idtoken을 사용해서 오류 발생 acesstoken 사용해서 해결
 - 챂에 bareer 붙여야함
 - cors설정등등

# AI가 정리한 AWS Cognito와 Unity 통합 가이드

## 목차
1. CloudFront 개요
2. Cognito 사용자 풀과 자격 증명 풀
3. 도메인 설정과 DNS
4. Google OAuth 설정
5. Unity 딥링크 구현
6. TCP 통신

## 1. CloudFront 개요
CloudFront는 AWS의 CDN 서비스입니다.

<div class="code-block-container">
  <pre><code>
기본 기능:
✅ 전세계 엣지 로케이션에 콘텐츠 캐싱
✅ HTTPS 제공 (무료 SSL 인증서)
✅ DDoS 보호
✅ 웹 애플리케이션 방화벽(WAF) 통합

일반적인 사용 사례:
S3 + CloudFront 조합:
└── S3: 원본 파일 저장
└── CloudFront: 
    ├── 캐싱으로 빠른 로딩
    ├── HTTPS 제공
    └── 보안 계층 추가
  </code></pre>
</div>

## 2. Cognito 사용자 풀과 자격 증명 풀

### 2.1 사용자 풀 (User Pool)
<div class="code-block-container">
  <pre><code>
목적: 사용자 인증 (Authentication)
기능:
- 회원가입/로그인
- 비밀번호 관리
- MFA
- 소셜 로그인
- 사용자 프로필

반환: JWT 토큰
  </code></pre>
</div>

### 2.2 자격 증명 풀 (Identity Pool)
<div class="code-block-container">
  <pre><code>
목적: AWS 리소스 접근 권한 부여 (Authorization)
기능:
- AWS 임시 자격증명 발급
- IAM 역할 매핑
- 인증/미인증 사용자 구분
- AWS 서비스 직접 접근

반환: AWS 임시 자격증명
  </code></pre>
</div>

## 3. 도메인 설정과 DNS

### 3.1 Route 53 설정
<div class="code-block-container">
  <pre><code>
A 레코드 설정:
Type: A Record
Name: yeougil.com
Value: 
  - Alias to API Gateway
  - 리전 선택
  - API Gateway 도메인 선택
  </code></pre>
</div>

### 3.2 API Gateway 커스텀 도메인
<div class="code-block-container">
  <pre><code>
1. API Gateway 콘솔
2. 사용자 지정 도메인 이름
3. "사용자 지정 도메인 생성" 클릭
4. 설정:
   - 도메인 이름: yeougil.com
   - 엔드포인트 유형: 리전형
   - ACM 인증서 선택
   - API 매핑 설정
  </code></pre>
</div>

## 4. Google OAuth 설정

### 4.1 리디렉션 URI 설정
<div class="code-block-container">
  <pre><code>
Authorized redirect URIs:
https://ap-northeast-2k75aczvmp.auth.ap-northeast-2.amazoncognito.com/oauth2/idpresponse

Cognito 앱 클라이언트 설정:
콜백 URL:
com.yeougil.crushcorecollapse://callback
  </code></pre>
</div>

## 5. Unity 딥링크 구현

### 5.1 Android 설정
<div class="code-block-container">
  <pre><code>
<!-- Android Manifest -->
<activity>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data 
            android:scheme="com.yeougil.crushcorecollapse"
            android:host="callback" />
    </intent-filter>
</activity>
  </code></pre>
</div>

### 5.2 iOS 설정
<div class="code-block-container">
  <pre><code>
<!-- iOS Info.plist -->
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>com.yeougil.crushcorecollapse</string>
        </array>
    </dict>
</array>
  </code></pre>
</div>

## 6. TCP 통신

### 6.1 C# TCP 클라이언트 구현
<div class="code-block-container">
  <pre><code>
public class TCPClient : MonoBehaviour
{
    private TcpClient client;
    private NetworkStream stream;
    private bool isRunning = true;

    private async void Start()
    {
        client = new TcpClient();
        await client.ConnectAsync("서버주소", 포트);
        stream = client.GetStream();
        
        // 수신 대기 시작
        _ = ReceiveLoop();
    }

    private async Task ReceiveLoop()
    {
        byte[] buffer = new byte[1024];
        
        while (isRunning)
        {
            try
            {
                int bytesRead = await stream.ReadAsync(buffer, 0, buffer.Length);
                if (bytesRead == 0) break;

                string message = Encoding.UTF8.GetString(buffer, 0, bytesRead);
                Debug.Log($"받은 메시지: {message}");
                
                ProcessMessage(message);
            }
            catch (Exception e)
            {
                Debug.LogError($"수신 에러: {e.Message}");
                break;
            }
        }
    }

    private void OnDestroy()
    {
        isRunning = false;
        client?.Close();
    }
}
  </code></pre>
</div>

## 주의사항 및 문제해결

### Invalid challenge transition 오류
이 오류는 Cognito Custom Domain 설정 과정에서 도메인 소유권 확인이 실패했을 때 발생합니다.

<div class="code-block-container">
  <pre><code>
확인사항:
1. ACM 인증서:
   - us-east-1 리전에 있어야 함
   - auth.yeougil.com 도메인 포함되어야 함
   - 검증 완료 상태여야 함

2. Route 53:
   - yeougil.com에 대한 A 레코드 존재
   - 올바른 대상으로 연결

3. 웹서버:
   - HTTP/HTTPS 응답 필요
   - 정상 작동 확인
  </code></pre>
</div>

### EC2 웹서버 설정
<div class="code-block-container">
  <pre><code>
# Amazon Linux 2 nginx 설치
sudo amazon-linux-extras install nginx1

# nginx 시작
sudo systemctl start nginx

# 자동 시작 설정
sudo systemctl enable nginx

# 상태 확인
sudo systemctl status nginx
  </code></pre>
</div>
### SSL/TLS 설정
<div class="code-block-container">
  <pre><code>
# SSL 디렉토리 생성
sudo mkdir -p /etc/nginx/ssl

# 인증서 및 키 생성
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/nginx-selfsigned.key \
  -out /etc/nginx/ssl/nginx-selfsigned.crt

# nginx SSL 설정
server {
    listen 443 ssl;
    server_name your_domain_or_IP;

    ssl_certificate /etc/nginx/ssl/nginx-selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx-selfsigned.key;

    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
}
  </code></pre>
</div>

## 보안 그룹 설정

### EC2 인바운드 규칙
<div class="code-block-container">
  <pre><code>
인바운드 규칙:
  - 유형: HTTP
    프로토콜: TCP
    포트 범위: 80
    소스: 0.0.0.0/0

  - 유형: HTTPS
    프로토콜: TCP
    포트 범위: 443
    소스: 0.0.0.0/0

  - 유형: SSH
    프로토콜: TCP
    포트: 22
    소스: 0.0.0.0/0 (또는 특정 IP)
  </code></pre>
</div>

## URL 인코딩 이해
<div class="code-block-container">
  <pre><code>
URL 인코딩 예시:
%3A = : (콜론)
%2F = / (슬래시)

예시:
원본 URL:
com.yeougil.crushcorecollapse://callback

URL 인코딩된 버전:
com.yeougil.crushcorecollapse%3A%2F%2Fcallback
  </code></pre>
</div>

## 아키텍처 구성

### 권장 아키텍처
<div class="code-block-container">
  <pre><code>
Route 53
    │
    ├── CloudFront (ACM 인증서 사용)
    │       │
    │       └── EC2 (nginx, HTTP만 처리)
    │
    └── Cognito Custom Domain

장점:
✅ ACM 인증서 자동 갱신
✅ CloudFront에서 SSL 종단
✅ EC2는 HTTP만 처리
✅ CDN 캐싱 효과
  </code></pre>
</div>

## Unity TCP 통신 구현

### 메시지 큐 처리
<div class="code-block-container">
  <pre><code>
public class MessageQueue : MonoBehaviour
{
    private Queue<string> messageQueue = new Queue<string>();
    private object lockObject = new object();

    public void EnqueueMessage(string message)
    {
        lock (lockObject)
        {
            messageQueue.Enqueue(message);
        }
    }

    private void Update()
    {
        if (messageQueue.Count > 0)
        {
            string message;
            lock (lockObject)
            {
                message = messageQueue.Dequeue();
            }
            ProcessMessage(message);
        }
    }

    private void ProcessMessage(string message)
    {
        // UI 업데이트 등 메인 스레드 작업
    }
}
  </code></pre>
</div>

### 이벤트 기반 TCP 클라이언트
<div class="code-block-container">
  <pre><code>
public class TCPClientEvent
{
    private Socket socket;
    private readonly SocketAsyncEventArgs receiveEventArgs;

    public TCPClientEvent()
    {
        socket = new Socket(AddressFamily.InterNetwork, 
                          SocketType.Stream, 
                          ProtocolType.Tcp);
        receiveEventArgs = new SocketAsyncEventArgs();
        receiveEventArgs.Completed += OnReceiveCompleted;
        receiveEventArgs.SetBuffer(new byte[1024], 0, 1024);
    }

    private void OnReceiveCompleted(object sender, SocketAsyncEventArgs e)
    {
        if (e.BytesTransferred > 0 && e.SocketError == SocketError.Success)
        {
            string message = Encoding.UTF8.GetString(e.Buffer, 
                                                   0, 
                                                   e.BytesTransferred);
            Debug.Log($"받은 메시지: {message}");

            bool willRaiseEvent = socket.ReceiveAsync(e);
            if (!willRaiseEvent)
            {
                OnReceiveCompleted(socket, e);
            }
        }
    }
}
  </code></pre>
</div>

## 문제 해결 가이드

### DNS_PROBE_FINISHED_NXDOMAIN 오류
<div class="code-block-container">
  <pre><code>
확인사항:
1. Route 53 설정:
   - A 레코드가 올바른지
   - 네임서버 설정이 올바른지
   - DNS 전파 대기 (최대 48시간)

2. 도메인 등록 대행사:
   - 네임서버가 Route 53과 일치하는지
   - 도메인 만료 여부

3. EC2/CloudFront:
   - 서비스가 정상 작동하는지
   - 보안 그룹 설정이 올바른지
  </code></pre>
</div>

### ERR_CONNECTION_REFUSED 오류
<div class="code-block-container">
  <pre><code>
확인사항:
1. 웹 서버 상태:
   sudo systemctl status nginx

2. 포트 리스닝:
   sudo netstat -tulpn | grep -E ':80|:443'

3. 보안 그룹:
   - 인바운드 규칙 80, 443 포트 확인

4. 로그 확인:
   sudo tail -f /var/log/nginx/error.log
  </code></pre>
</div>

## 성능 최적화

### API Gateway 응답 시간 개선
<div class="code-block-container">
  <pre><code>
최적화 방법:
1. Lambda 최적화:
   - 프로비저닝된 동시성 설정
   - 메모리 증가
   - 웜업 설정

2. 캐싱 설정:
   - API Gateway 캐싱 활성화
   - CloudFront 배포 사용

3. 모니터링:
   - CloudWatch 로그 확인
   - X-Ray 추적 활성화
  </code></pre>
</div>

## 보안 고려사항

### SSL/TLS 설정
<div class="code-block-container">
  <pre><code>
권장 설정:
1. CloudFront에서 SSL 종단
2. ACM 인증서 사용 (자동 갱신)
3. EC2는 내부 통신만 처리
4. 보안 그룹으로 접근 제한
  </code></pre>
</div>

# 또다른 AI 정리

# AWS Cognito Unity 구현 가이드

## 개요
Unity 게임에서 AWS Cognito를 이용한 인증 시스템 구현 과정을 정리합니다. 특히 소셜 로그인과 딥링크를 통한 인증 흐름에 대해 자세히 다룹니다.

## 1. 딥링크와 리다이렉트 URL 이해

### 딥링크 URL 형식
<div class="code-block-container">
com.yeougil.crushcorecollapse://callback
</div>

### Cognito 호스팅 UI URL 형식
<div class="code-block-container">
https://[your-domain].auth.[region].amazoncognito.com/login?client_id=[your-client-id]&redirect_uri=[your-redirect-uri]&response_type=code&scope=email+openid+phone&identity_provider=Google
</div>

## 2. 플랫폼별 설정

### Android 설정
AndroidManifest.xml에 추가:
<div class="code-block-container">
<manifest>
    <application>
        <activity android:name="com.unity3d.player.UnityPlayerActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data 
                    android:scheme="com.yeougil.crushcorecollapse"
                    android:host="callback" />
            </intent-filter>
        </activity>
    </application>
</manifest>
</div>

### iOS 설정
Info.plist에 추가:
<div class="code-block-container">
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>com.yeougil.crushcorecollapse</string>
        </array>
    </dict>
</array>
</div>

### Windows 개발 환경 설정
Windows에서는 딥링크 대신 localhost를 사용합니다:
<div class="code-block-container">
#if UNITY_EDITOR || UNITY_STANDALONE_WIN
    private const string RedirectUrl = "http://localhost:8080/callback";
#else
    private const string RedirectUrl = "com.yeougil.crushcorecollapse://callback";
#endif
</div>

## 3. 인증 흐름 구현

### 1. 로그인 URL 생성
<div class="code-block-container">
public string GetLoginUrl()
{
    return "https://" + AuthCognitoDomainPrefix + "/login"
        + "?client_id=" + AppClientID
        + "&redirect_uri=" + UnityWebRequest.EscapeURL(RedirectUrl)
        + "&response_type=code"
        + "&scope=email+openid+phone"
        + "&identity_provider=Google";
}
</div>

### 2. 인증 코드 수신 처리
모바일과 Windows에서 각각 다른 방식으로 처리:

**모바일 (딥링크)**:
<div class="code-block-container">
private void Awake()
{
    Application.deepLinkActivated += OnDeepLinkActivated;
    if (!string.IsNullOrEmpty(Application.absoluteURL))
    {
        OnDeepLinkActivated(Application.absoluteURL);
    }
}

private void OnDeepLinkActivated(string url)
{
    ExchangeAuthCodeForAccessToken(url);
}
</div>

**Windows (로컬 웹서버)**:
<div class="code-block-container">
private async void StartLocalServer()
{
    _httpListener = new HttpListener();
    _httpListener.Prefixes.Add("http://localhost:8080/");
    _httpListener.Start();
    
    while (true)
    {
        var context = await _httpListener.GetContextAsync();
        var code = context.Request.QueryString["code"];
        if (!string.IsNullOrEmpty(code))
        {
            await ExchangeAuthCodeForAccessToken($"http://localhost:8080/callback?code={code}");
            // 브라우저에 성공 메시지 표시
            var response = context.Response;
            string responseString = "<html><body>인증 성공! 이 창을 닫아도 됩니다.</body></html>";
            byte[] buffer = System.Text.Encoding.UTF8.GetBytes(responseString);
            response.ContentLength64 = buffer.Length;
            response.OutputStream.Write(buffer, 0, buffer.Length);
            response.OutputStream.Close();
        }
    }
}
</div>

### 3. 토큰 교환
<div class="code-block-container">
private async Task<bool> CallCodeExchangeEndpoint(string grantCode)
{
    WWWForm form = new WWWForm();
    form.AddField("grant_type", AuthCodeGrantType);
    form.AddField("client_id", AppClientID);
    form.AddField("code", grantCode);
    form.AddField("redirect_uri", RedirectUrl);

    string requestPath = "https://" + AuthCognitoDomainPrefix + TokenEndpointPath;
    UnityWebRequest webRequest = UnityWebRequest.Post(requestPath, form);
    webRequest.SetRequestHeader("Content-Type", "application/x-www-form-urlencoded");
}
</div>

## 4. AWS Cognito 설정

### 앱 클라이언트 설정
1. Allowed callback URLs 등록
   - 모바일: `com.yeougil.crushcorecollapse://callback`
   - Windows 개발: `http://localhost:8080/callback`
2. OAuth 2.0 설정
   - Grant type: Authorization code grant
   - Scopes: email, openid, phone

### 호스팅 UI 설정
1. 도메인 이름 설정
2. ID 공급자 설정 (Google 등)
3. UI 커스터마이징 (선택사항)

## 5. 주의사항

### 리다이렉트 URI 일치
- Cognito 콘솔에 등록된 URI
- 코드 수신 시 사용하는 URI
- 토큰 교환 시 사용하는 URI
모두 정확히 일치해야 합니다 (대소문자, 슬래시, http/https 포함).

### Content-Type 헤더
토큰 교환 시 반드시 `application/x-www-form-urlencoded`를 사용해야 합니다.

### 플랫폼별 차이
- 모바일: 딥링크 사용
- Windows: localhost 웹서버 사용

### 보안
- 인증 코드는 일회성이며 짧은 시간 동안만 유효
- 토큰 교환 시 client_id 검증 필요
- HTTPS 사용 권장

## 6. 디버깅

### 로그 추가
<div class="code-block-container">
Debug.Log($"Request URL: {requestPath}");
Debug.Log($"Redirect URI: {RedirectUrl}");
Debug.Log($"Grant Code: {grantCode}");
Debug.Log($"Response: {webRequest.downloadHandler.text}");
</div>

### 일반적인 오류
1. 400 Bad Request
   - 리다이렉트 URI 불일치
   - Content-Type 헤더 누락
   - 잘못된 grant_type

2. 401 Unauthorized
   - 잘못된 client_id
   - 만료된 인증 코드

## 7. 용어 설명

- **딥링크**: 앱을 직접 실행하는 URL 스킴
- **리다이렉트 URI**: 인증 완료 후 돌아갈 주소
- **인증 코드**: 임시로 발급되는 인증 증명
- **토큰**: 실제 API 호출에 사용되는 인증 정보

## 결론
AWS Cognito를 Unity에 구현할 때는 플랫폼별 차이를 고려해야 합니다. 특히 Windows 개발 환경과 모바일 환경의 차이를 이해하고, 적절한 방식으로 처리하는 것이 중요합니다. 또한 보안을 위해 HTTPS 사용과 토큰 관리에 주의를 기울여야 합니다.



## 참고 자료
- [AWS Cognito 개발자 가이드](https://docs.aws.amazon.com/cognito)
- [Unity WebGL 보안 가이드](https://docs.unity3d.com/Manual/webgl-security.html)
- [AWS 아키텍처 모범 사례](https://aws.amazon.com/architecture/well-architected)
