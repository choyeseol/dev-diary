## HTTPS?
HTTPS: **하이퍼 텍스트 전송 프로토콜 보안**(Hyper Text Transfer Protocol Secure)
- **웹사이트가 SSL/TLS 인증서로 보호**되는 경우, HTTPS가 URL에 표시된다.
사용자는 브라우저 표시줄의 자물쇠 기호를 클릭해 발급 기관 및 웹사이트 소유자의 상호를 포함한 인증서의 세부 정보를 볼 수 있다.

### HTTPS를 적용시켜야 하는 이유는 무엇일까?
1. **보안적인 이유**
데이터를 서버와 주고 받을 때 암호화를 시켜서 통신한다. 
암호화를 하지 않으면 누군가가 중간에 데이터를 가로채서 해킹을 시도할 수 있다.
	_* 내가 보내는 정보를 다른 사람이 보지 못하게 한다._

2. **사용자 이탈**
어떤 사이트에 들어갔는데, 아래와 같이 보인다면 _**'뭔가 껄끄러운 사이트...'**_라고 생각할 것이다.
![](https://velog.velcdn.com/images/choyeseol/post/1d394198-135a-4495-9043-42e506f5617e/image.png)

주로 도메인을 구성할 때, 아래와 같이 구성한다.
- 웹 사이트 주소: `http://www.도메인.com`
- 백엔드 API 서버 주소: `http://api.도메인.com`

### 대칭키와 비대칭 키
![](https://velog.velcdn.com/images/choyeseol/post/0392dc72-fab0-4e3f-bd0b-284e91afbf48/image.png)

1. **대칭키**
동일한 키를 사용해서 암호화 복호화 가능

2. **비대칭키(공개키)**
내가 보내는 정보를 중간에서 훔쳐볼 수 없도록!
	_* 서버측은 어떻게 자신임을 증명하는가?_
    ![](https://velog.velcdn.com/images/choyeseol/post/8d8bdd3e-f334-4e5e-aa32-d0d6e1c0a54c/image.png)

3. **CA(Certificate Authority)** 
→ 네이버가 뿌린 공개키가 정품인지 인증해주는 민간 기업
	브라우저에서 CA 목록이 내장되어 있다.
    클라이언트와 서버의 통신 과정

## SSL/TLS란?  
쉽게 표현하자면, **HTTP를 HTTPS로 바꿔주는 인증서**이다.  SSL/TLS 인증서를 활용해 HTTP가 아닌, HTTPS로 통신할 수 있게 만들어준다.
- **SSL**: 보안 소켓 계층(Swcure Sockets Layer)
	웹사이트와 브라우저 사이(또는 두 서버 사이)에 **전송되는 데이터를 암호화**하여 인터넷 연결을 보호하기 위한 표준 기술
- **TLS**: 전송 계층 보안(Transport Layer Security)
	TLS는 SSL의 향상된, 더욱 안전한 버전이다.



# SSL 인증서 발급받기 (ubuntu)

### 1. Nginx 설치
``` 
sudo apt update
sudo apt install nginx
```

### 2. Nginx 설치 확인
```
systemctl status nginx
```
**EC2 IP로 접속해서 확인**
![](https://velog.velcdn.com/images/choyeseol/post/592dff8e-e663-4356-ac9a-e53dbaa7738e/image.png)
주의> `https://<ip 주소>`가 아니라, 반드시 `http://<ip 주소>`로 접속해야한다.

### 3. Cerbot 설치하기
```
#Certbot 설치
sudo snap install --classic certbot

#명령 준비
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 5. SSL 인증서 발급받기
```
sudo certbot --nginx
# 이메일 입력
# Y
# Y
# [도메인]

sudo certbot --nginx -d [도메인]

#인증서확인
sudo certbot certificates
```
`sudo certbot --nginx`이 명령어를 치고, 도메인을 입력했을 때 아래 에러가 뜬다면 DNS 설정을 하지 않아 도메인 확인에 실패한 것이다. (DNS 설정 먼저 헤야함)
```
Certbot failed to authenticate some domains (authenticator: nginx). 
The Certificate Authority reported these problems:  
  Domain: moddo.kro.kr
  Type:   dns
  Detail: DNS problem: NXDOMAIN looking up A for   moddo.kro.kr - check that a DNS record exists for this  
domain; DNS problem: NXDOMAIN looking up AAAA for 
moddo.kro.kr - check that a DNS record exists for this domain

Hint: The Certificate Authority failed to verify the 
temporary nginx configuration changes made by Certbot. 
Ensure the listed domains point to this nginx server and that it is accessible from the internet.

Some challenges have failed.
Ask for help or search for solutions at https://community.letsencrypt.org. See the logfile 
/var/log/letsencrypt/letsencrypt.log or re-run Certbot 
with -v for more details.
```


### 6. 리버스 프록시 설정 해주기
```
sudo vi /etc/nginx/sites-available/default
```
전부 지우고 아래와 같이 수정함.
```
# HTTP -> HTTPS 리다이렉트
server {
    listen 80;
    listen [::]:80;
    server_name moddo.kro.kr;

    location /.well-known/acme-challenge/ {
         root /var/www/certbot;  # 인증 파일을 저장할 경로
     }

    return 301 https://$host$request_uri;
}

# HTTPS 설정
server {
    listen 443 ssl;
    listen [::]:443 ssl ipv6only=on;
    server_name moddo.kro.kr;

    ssl_certificate /etc/letsencrypt/live/moddo.kro.kr/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/moddo.kro.kr/privkey.pem;

    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # 기본 웹사이트 설정 (백엔드 서버로 프록시)
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 7. Nginx 재시작하기
```
sudo systemctl restart nginx
```

백엔드 서버에 HTTPS가 잘 적용되는 지 확인하기