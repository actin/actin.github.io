---
layout : post
title : ssh 인증키 생성 및 서버 등록
tags : tip
typora-root-url: ..
---



# PEM 이란?

PEM (Privacy Enhanced Mail)은 Base64 로 인코딩한 텍스트 형식의 파일입니다.

Binary 형식의 파일을 전송할 때 손상될 수 있으므로 TEXT 로 변환하며 소스 파일은 모든 바이너리가 가능하지만 주로 인증서나 개인키가 됩니다.



2048 비트의 RSA 키를 생성합니다.

```
ssh-keygen -t rsa -b 2048 -m pem
```



```
cd ~/.ssh
#authorized_keys 파일이 없는 경우 생성
touch authorized_keys

```



# ssh config로 ssh 접속 간편하게 하기

`~/.ssh/config` 의 설정을  통해 간편하게 접속이 가능 합니다.

```
Host svr_test
	HostName 192.168.1.1
	User ubuntu
	PreferrAuthentications publickey
	IdentityFfile ~/.ssh/my_ssh_key.pem
```



