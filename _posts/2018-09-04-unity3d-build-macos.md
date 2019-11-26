---
layout: post
title: "Jenkins Unity3D 빌드하기 on MacOS"
categories: devlog
tags: macos
toc: true
typora-root-url: ..
---

MacOS 에서 Jenkins 설치는 [Jenkins 설치 on MacOS]({% post_url 2018-07-30-macos-jenkins %})  를 참고하세요.  

진행하기 앞서 MacOS **Xcode** , **Unity3D** 는  먼저 설치가 되어 있어어야 합니다. 
이 부분은 여기서 다루지 않겠습니다.

## 준비하기 - plugin 설치

[Jenkins 관리] => [플러그인 관리] 에서 Unity3D, xcode 플러그인을 설치 하도록 합니다.

* Xcode integration
* Unity3d plugin
해당 플러그인이 설치가 정상적이라면,  다음 단계  설정 부분으로 넘어갑니다.

## Unitdy3D 설정

[Jenkins 관리] => [Global Tool Configuration] 들어가자 Unity3D 설치 경로를 입력합니다.

기본 설치를 진행 하였다면, 입력된 경로에 설치 되어 있을 것입니다.
{: .notice--info}

![1536163078204](/assets/mac-unity-build-01.png)



## Jenkins 프로젝트 설정

[Jenkins] 의 [새로운 item] 메뉴를 이용하여, 프로젝트를 구성을 합니다.
기본적인 General , 소스 코드 관리 프로젝트에 맞게 설정 하시면 됩니다.



## Build-Invoke Unity3d Editor

[Add Build Step] 메뉴에서 [Invoke Unity3d Editor] 항목 선택합니다.

- Unity3d installation name  : 
   Unity3d 설정에서 입력한 항목을 선택 합니다.
- Editor command line arguments
  유니티 콘솔 빌드시 입력되는 항목입니다.

  ```
  -quit -batchmode  -stackTraceLogType Full -projectPath "$WORKSPACE" -buildTarget ios -executeMethod NUnityBuilder.BatchRun GNSS.ios.qa.ini -logFile $WORKSPACE/build_unitylog.txt
  ```

![1536163014173](/assets/mac-unity-build-07.png)

빌드 옵션은 Unity3d 문서를 참고 하시면 자세한 내용을 확인 할 수 있습니다.
  {: .notice--info}

  

## Build - Xcode

[Add Build Step] 메뉴에서 [xcode] 항목을 선택합니다. 그러면 아래와 같은 항목이 추가 됨을  확인 할 수 있습니다. 추가 내용에 대해서는 아래의 스텝으로 설정 하시면 됩니다.

![1536162685234](/assets/mac-unity-build-06.png)



### General build settings

* Development Team ID :  애플 개발자 사이트에서 발급된 아이디를 입력합니다.
* Target :  Unity 빌드시 생성된 xcode 프로젝트의 Target 을 입력합니다. 기본적으로 Unity-iPhone 으로 설정 되어 있습니다.
* Configuration : Release 입력합니다.
* Xcode Schema File : Target 에 입력한 내용을 입력 합니다.

![1536161595719](/assets/mac-unity-build-02.png)

* Export method : 배포를 위한 방법을 선택합니다. 개발자 배포 관련 항목은 사업팀 혹은 플랫폼팀으로 문의 하시면 됩니다. 
* .ipa filename pattern : 생성될 ipa 파일의 이름을 지정합니다.

![1536161882553](/assets/mac-unity-build-03.png)
* Application URL : manifest.plist 파일의 [software-package]  항목에 사용되는 URL 입니다. 

* Display image URL : manifest.plist 파일의 [display-image]  항목에 사용되는 URL 입니다.

* Full size image URL : manifest.plist 파일의 [full-size-image]  항목에 사용되는 URL 입니다.
  

**image 파일은 꼭 해당 주소에 존재 하지 않아도 됩니다. 하지만 항목이 누락되면. 빌드시 실패 할수 있으니. 확인하시고 입력 해주시면 됩니다.**



![1537186825330](/assets/mac-unity-build-08.png)

### Code signing & OS X keychain options

* Bundle ID : 
* Provisioning profile UUID : 
* Keychain path :
* Keychain password : 

![1536162101035](/assets/mac-unity-build-04.png)

### Advanced Xcode build options

* Xcode Project Directory : xcode 프로젝트 파일이 위치한 젠킨스 프로젝트 작업디렉토리를 기준으로 상대 경로를 입력합니다.
* Build output directory : 빌드 후 결과물이 저장 되는 경로입니다.

![1536162459493](/assets/mac-unity-build-05.png)






