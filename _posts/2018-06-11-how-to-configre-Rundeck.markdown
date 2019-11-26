---
layout: post
title:  "Rundeck 설정하기"
date:   2018-06-01 23:53:00
categories: devlog
tags: rundeck
toc: true
typora-root-url: ..
---


## 다운로드
[https://www.rundeck.com/open-source](https://www.rundeck.com/open-source)

## 설치하기
java 8이 설치 되어 있는지 확인을 합니다. 최신 버전의 자바를 설치시 rundeck 설치가 안 될수 있습니다.

설치 파일을 복사한 후 아래의 명령을 실행합니다.
```
c:\rundeck> java -jar rundeck-launcher-2.11.3.jar -d
```
명령을 실행하면 아래의 이미지와 같은 폴더가 생성되어야 합니다. 아래와 같은 폴더가 생성되지 않았다면, 설치시 문제가 발생하였을 것입니다. 

![1536242386748](/assets/rundeck04.png)



etc\profile.bat 파일을 열어 사용 메모리수치를 수정합니다.

```
set RDECK_CLI_OPTS=-Xms512m -Xmx1024m
```



## Rundeck 을 Windows Service 로 등록하기

NSSM  (the Non-Sucking Service Manager) 을 이용하여 등록하자  [다운로드 ](https://nssm.cc)

윈도우즈 커맨드 창에 nssm install [서비스명] 을 입력하면 아래와 같은 GUI 화면을 볼수 있다.

```
c:\rundeck>nssm install rundeck
```

![1536048864250](/assets/rundeck03.png)


* start-rundeck.bat 을 아래와 같이 작성한다.

```
set CURDIR=%~dp0
call %CURDIR%etc\profile.bat
java %RDECK_CLI_OPTS% %RDECK_SSL_OPTS% -jar rundeck-launcher-2.11.3.jar --skipinstall -d
```

* 방화벽 오픈 
  윈도우는 기본적으로 방화벽 기능이 활성화 되어 있으므로. 서비스를 위해서. 방화벽 오픈이 필요합니다. 윈도우 명령창에서 아래의 명령을 입력하세요.
  
```
방화벽 오픈
netsh advfirewall firewall add rule name="rundeck Http Port : 4440" dir=in action=allow protocol=TCP localport=4440
```

  



## Rundeck node 추가

노드는 프로젝트에서 사용될 배포하게될 해당 머신을 뜻하며, 노드 추가를 위해선 먼저 프로젝트가 생성되어야 합니다. 프로젝트가 생성되어 있다면, 아래의 경로에 파일을 수정하여 노드를 추가 합니다.

[rundeck 설치 경로]\\projects\\[프로젝트명]\\[etc]\\resources.xml 

```xml
<node name="GAME-01"
    connectionType="WINRM_NATIVE"
    node-executor="overthere-winrm"
    winrm-password-option="winrmPassword"
    winrm-protocol="http"
    winrm-auth-type="basic"
    username="접속 아이디"
    winrmPassword="암호"
    description="ZK Game sever"
    tags="game"
    hostname="10.0.0.228:5985"
    osArch="x86_64"
    osFamily="windows"
    osName="Microsoft Windows Server 2016"
    osVersion="Microsoft Windows Server 2016" />
```

**name** : 노드를 표현하는 이름입니다. 배포 구성시 필터에 사용됩니다.

**tags** : 배포 구성시 필터에 사용되며, 구성을 표현할 수 있는 키워드라고 생각하시면 됩니다.



## winrmPassword

현재 버전에서는 node 정보에 wirm-password-option 을 이용하여 암호 설정을 하여도 적용 되지 않는 문제가 있습니다.

오른쪽 상단의 설정에서 **[Key Storage]** 메뉴를 이용하여 암호를 설정합니다.

![](/assets/rundeck01.png)


프로젝트 설정에서 [Default Node Executor] -> [WinRM] 옵션을 활성화 후 아까 생성한 키를 선택합니다.

![](/assets/rundeck02.png)



## 윈도우 노드의 winrm 설정

Winrm 이란 Windows 원격 관리 명령줄 도구이며, Microsoft가 구현한 WS-Management 프토콜로서, 웹 서비스를 이용하여 로컬 및 원격 컴퓨터와 안전하게 통신을 할 수 있는 방법입니다.

윈도우 노드 설정

```powershell
winrm qc
winrm set winrm/config/client/auth @{Basic="true"}
winrm set winrm/config/service/auth @{Basic="true"}
winrm set winrm/config/service @{AllowUnencrypted="true"}
```

**Windows server 2012** 인 경우 powershell 실행 권한이 필요합니다.
{: .notice--info}

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```


