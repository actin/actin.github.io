---
layout: post
title: "Jenkins 설치 하기 on MacOS"
categories: devlog
tags: macos
toc: true
typora-root-url: ..
---


# MacOS Jenkins 설치하기

homebrew 를 이용하여 쉽게 설치가 가능합니다.

## Homebrew 설치 하기

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```



```bash
$brew install jenkins
```



유저 계정으로 다시 실행하기

```bash
$launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
$launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
$brew services stop jenkins
```



젠킨스를 설치하고 실행후 접속해보자

http://127.0.0.1:8080 또는 http://localhost:8080 으로 접속 할 수 있다.



## Jenkins 설정하기

초기 설치 암호는 화면의 경로에 있는 파일을 열어 입력할 수 있다.
화면을 따라 설치 하면 어렵지 않게 설정을 진행 할 수 있다.



로컬 호스트 이외의  도메인이나 IP로 접속하기 위해선 다음 경로의 파일을 수정해야한다.
{: .notice--info}

```shell
$ cd /usr/local/opt/jenkins
$ vi homebrew.mxcl.jenkins.plist
```



파일에서 127.0.0.1 부분을  **0.0.0.0** 으로 변경 후 저장한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
 
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>homebrew.mxcl.jenkins</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/bin/java</string>
      <string>-Dmail.smtp.starttls.enable=true</string>
      <string>-jar</string>
      <string>/usr/local/opt/jenkins/libexec/jenkins.war</string>
      <string>--httpListenAddress=127.0.0.1</string>
      <string>--httpPort=8080</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
  </dict>
</plist>



```

편집 후 젠킨스를 재시작 한다.

```shell
$ brew services restart jenkins
```





## Jenkins 제거

아래의 명령으로 간단하게 제거 가능하다.

```shell
$ brew services stop jenkins
$ brew remove jenkins
```

사용 되었던 데이터까지 모두 삭제를 원한다면, 계정의 홈디렉토리에서 .jenkins 디렉토리를 삭제 하면된다.

```shell
$ rm -rf ~/home_account/.jenkins
```



## Aws CLI 설치

참고 URL <https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-macos.html#install-bundle-macos>

Python Pip 로 설치 하는 방법이 있지만, 번들 설치 관리자를 사용 하여 설치합니다.

위 가이드대로 설치 한다면, aws 설치 경로는 **/usr/local/aws**  이 될 것입니다.









