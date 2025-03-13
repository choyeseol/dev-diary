## 1. 비밀번호 로그인 활성화하기

### 1️⃣ EC2에 현재 사용 중인 키 페어로 SSH 접속
``` ubuntu
ssh -i [pem 파일] ec2-user@[ec2 인스턴스 ip]
```

### 2️⃣ SSH 설정 파일 수정
파일에 접속
```ubuntu
sudo vi /etc/ssh/sshd_config
```
`PasswordAuthentication`를 `yes`로 변경
```ubuntu
- PasswordAuthentication no
+ PasswordAuthentication yes
```

### 3️⃣ SSH 서비스 재시작
```ubuntu
sudo systemctl restart ssh
```


ㅤ


## 2. 사용자 비밀번호 설정
### 1️⃣ ubuntu 사용자 비밀번호 설정

```ubuntu
sudo passwd [ec2 사용자 계정]
```

### 2️⃣ EC2 접속
```ubuntu
ssh ubuntu@[탄력적 IP 혹은 ec2 인스턴스 ip]
```

![](https://velog.velcdn.com/images/choyeseol/post/9b390089-e078-4a07-94f7-c08a2d9bb677/image.png)