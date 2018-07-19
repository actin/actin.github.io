---
title: "명령창 열기 Windows 10 context menu "
categories: program
tags: windows
toc: true
typora-root-url: ..
---



윈도우즈 10은 기본적으로 context menu 에서 '여기에서 명령창 열기' 가 사라지고 Powershell 열기가 생겼다.
많은 작업을 cmd 로 하는 중 점점 불편함을 느끼기 시작하여, 추가 하는 방법을 작성 한다.


# Context Menu 추가하기



**경고** - 레지스트리를 수정하기 때문에 반드시 백업후에 아래의 스텝을 수행하시기 바랍니다.
{: .notice--warning}



1. 윈두우키 + R 을 입력하여, 실행 창을 오픈한다.
2. regedit 을 입력하여 레지스트리 에디터를 실행한다.
3. 아래의 경로를 찾는다.
   ```
   HKEY_CLASSES_ROOT\Directory\shell\cmd
   ```
   
4. cmd 폴더를 마우스 우클릭 , 사용 권한을 클릭한다.
  
   ![1531270186812](/assets/cmd-context-1.png){: .full}
  
5. 고급 버튼을 클릭한다.
  
   ![1531270456051](/assets/cmd-context-2.png){: .full}

6. cmd 고급 보안 설정에서 **변경** 을 클릭한다.

   ![1531270638094](/assets/cmd-context-3.png)

7. 사용자 계정을 입력후 이름 확인을 눌러 확인후 확인 버튼을 누른다.

   ![1531270675420](/assets/cmd-context-4.png)

8. 하위 컨테이너와 개체의 소유자 바꾸기 항목을 체크후 적용 및 확인 버튼을 누른다.

   ![1531270788136](/assets/cmd-context-5.png)
   
   ![1531270810132](/assets/cmd-context-6.png)


9. cmd 의 **HideBasedOnVelocityId** 의 이름을 **ShowBasedOnVelocityId** 로 변경한다.

	![1531272384763](/assets/cmd-context-7.png)















