---
layout: post
title : "Rundeck 배포 구성하기"
categories : devlog
tags : infra
toc : true
typora-root-url: ..
---



배포 과정을 게임서버 기준으로 설명을 하도록 하겠습니다.

아래의 그림은 배포 과정을 간략하게 정리하였습니다.

![1539672269924](assets/rundeck-deplay-12.png)

## 배포를 위한 준비

* 각 노드들은 WinRM 설정이 되어 rundeck 과 연동이 되는지 확인이 필요합니다.

  프로젝트의 [commands] 메뉴의 기능으로 각 설정 되어 있는 노드의 상태를 확인합니다. 
  ![1537258291633](assets/rundeck-deplay-03.png)

  연결이 정상적이지 않다면, 방화벽이 오픈 되어 있는지, 접속 계정 아이디,암호가 옳바른지 확인 합니다.

* AWS command CLI 설치
  Windows [64비트](https://s3.amazonaws.com/aws-cli/AWSCLI64.msi), [32비트](https://s3.amazonaws.com/aws-cli/AWSCLI32.msi)
*  S3 cli 사용을 위한 설정이 필요합니다. 윈도우 커맨드창을 실행한 후 aws configure 를 실행하여, Access Key 와 Secret Access Key 를 입력 하시면 됩니다.

  ```powershell
  c:\> aws configure
  AWS Access Key ID :
  AWS Secret Access Key :
  Default region name [] :
  Default output format [] : 
  ```


* windows server 2012 인경우 Powershell 실행 권한 부여

  ```powershell
  ps>Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
  ```



rundeck 배포를 위해서는 배포용 파일, 배포를 위한 버전 파일 의 두개 파일이 필요 합니다. 

**배포용 파일**  :  빌드 시스템에서 배포용을 생성되는 파일입니다.  파일명  규칙은 특별히 지정된건 없습니다.  다만 파일명은 배포 버전 파일에 기록에 사용됩니다. 
​	예> GNSS_Game_20180402_1102.zip

**배포 버전 파일** : 업로드 된 배포용 파일중  현재 배포를 해야 하는 파일명을 가지고 있습니다.

```xml
build_game.xml

<?xml version="1.0" encoding="utf-8"?> 
<File>GNSS_Game_20180402_1102</File> 

```



## 배포 파일 생성 및 버전 파일 생성

**%DEPLOY_GAME%** : 스크립트를 이용하여 배포 파일이 생성되는 경로

**%OUTPUT_GAME%** : 젠킨스를 이용하여 빌드된 결과 파일과 배포에 필요한 파일이 있는 경로

아래 내용은 젠킨스에서 Execute Windows batch command 항목의 내용입니다.

```
:: GAME 압축 및 S3 업로드 작업
:: %DEPLOY_GAME% 
:: %OUTPUT_GAME%
:: 압축할 파일 이름에 넣을 날짜, 시간를 설정합니다.
for /f "tokens=1,2 delims=:" %%a in ("%time%") do set hh=%%a&set mm=%%b
SET hh=%hh: =0%
:: 압축 파일 이름입니다.

SET FILENAME=Super_Game_%date:~0,4%%date:~5,2%%date:~8,2%_%hh%%mm%
:: 압축 파일 풀 경로
SET ZIPFILE=%DEPLOY_GAME%\%FILENAME%.zip

if not exist %DEPLOY_GAME%\NUL (
    md %DEPLOY_GAME%
)

:: 배포용 파일 압축
D:\Jenkins\script\7za.exe a %ZIPFILE% %OUTPUT_GAME%\*.* -r -x!*.pdb

:: 버전 파일 생성
SET BUILD_INFO=%DEPLOY_GAME%\build_game.xml
@ECHO Create build_game.xml.
@ECHO ^<?xml version="1.0" encoding="utf-8"?^> > %BUILD_INFO%
@ECHO ^<File^>%FILENAME%^</File^> >> %BUILD_INFO%

cd /D %DEPLOY_GAME%
:: 생성된 파일 S3 업로드
PowerShell.exe -executionpolicy remotesigned -Command D:\Jenkins\script\upload_game_s3.ps1 -file %ZIPFILE% -path %DEPLOY_GAME% -mode "qa"

```



## 배포를 위한 S3 폴더 구성

S3 버킷의 폴더 구성은 아래의 이미지와 같습니다.
/build 이하에 각 서버별 배포 파일을 보관 할 수 있는 폴더와, 각 서버별 버전 xml 파일로 구성 되어 집니다.

![1537256535052](/assets/rundeck-deplay-01.png)





## Rundeck Node(Windows) 폴더 구성

배포를 위해서는 각 노드들의 경로를 통일 시켜야 합니다. Rundeck은 각노드들의 일괄 명령 처리로 동작을 하기 때문에 서버 배포 위치를 아래의 그림과 같이 드라이버 폴더 구성을 통일 시켜야 합니다.

**/script** : 배포에 필요한 파일들이 위치합니다. 배포 스크립트, 압축 파일등

**/work** : 배포 스크립트에서 사용되며, 배포 파일을 임시로 다운로드 및 압축, 백업 등에 사용됩니다. 

**/Gnss_Game** : 게임 서버 배포가 되는 위치 입니다. 각 종 설정 파일과 게임에 필요한 파일들이 위치하며, 배포 스크립트에서 경로를 설정하면 자동 생성 됩니다.

**/Gnss_Auth** : 인증 서버 배포 되는 위치입니다. 게임서버와 동일하게 필요 파일들이 위치하며, 역시 스크립트에서 자동 생성됩니다.

![1537257386654](/assets/rundeck-deplay-02.png)

## 배포를 위한 스크립트 구성

deploy_build_game.ps1 파일의 내용입니다. 아래의 스크립트를. [script] 폴더에 생성한 후에 필요 항목들을 수정 하시면 됩니다.

```powershell
param([Parameter(Mandatory=$false)][string]$mode="qa" )

$ErrorActionPreference = "Stop"

$DEBUG_MODE = 1
# 1. Argument set #

$GAME_BUILDZONE = $mode
$GAME_BUILDTYPE = "Super_Game"

# S3 bucket set #
$AWS_S3_ADDR = "s3://super-build"
$AWS_S3_REGION = "us-east-1"

# 2. define variable #
write-output "."
write-output "[INFO][1] Define Variable..."
$TIME_CUR = Get-Date -UFormat '%Y-%m-%d %H:%M:%S'
write-output "[INFO] Start TIME : $TIME_CUR"
write-output "."

# 2-1. get hostname #
$HOSTNAME = hostname
# 2-2. get date / custom type #
$DATE_CUR = Get-Date -UFormat '%Y%m%d-%H%M'
$TIME_CUR = Get-Date -UFormat '%Y-%m-%d %H:%M:%S'

# 2-3. default PATH #
$DEF_FILE_INFO = "build_game.xml"
$DEF_FILE_INFO_SAMPLE = "build_game_sample.xml"
$DEF_PATH_BASE = "super"
$DEF_PATH_SCRIPT = "script"
#$DEF_PATH_SERVER = $GAME_BUILDTYPE
$DEF_PATH_BACKUP = Join-Path "work" "backup"
$DEF_PATH_WORK = Join-Path "work" "download"
$DEF_PATH_TMP = join-path "work" "tmp"

$DriveLetter = $PSCommandPath[0]

$PATH_BASE = $DriveLetter + ":\" + $DEF_PATH_BASE
$PATH_SCRIPT = join-path $PATH_BASE $DEF_PATH_SCRIPT

$PATH_SERVER =  join-path $PATH_BASE $GAME_BUILDTYPE
$PATH_SERVER_CONF = join-path $PATH_SERVER "conf"
$PATH_CONF = join-path $PATH_BASE "conf_game"

$PATH_BACKUP = join-path $PATH_BASE $DEF_PATH_BACKUP
$PATH_WORK = join-path $PATH_BASE $DEF_PATH_WORK
$PATH_WORK_SERVER = join-path $PATH_WORK $GAME_BUILDTYPE

$PATH_WORK_SERVER_CONF = $PATH_WORK_SERVER
$PATH_TMP = join-path $PATH_BASE $DEF_PATH_TMP

$FILE_BUILD_INFO = join-path $PATH_SERVER $DEF_FILE_INFO
if($DEBUG_MODE = 1) {
  write-output "[DEBUG] FILE_BUILD_INFO : $FILE_BUILD_INFO"
}
$FILE_BUILD_INFO_SAMPLE = join-path $PATH_SCRIPT $DEF_FILE_INFO_SAMPLE
if($DEBUG_MODE = 1) {
  write-output "[DEBUG] FILE_BUILD_INFO_SAMPLE : $FILE_BUILD_INFO_SAMPLE"
}

$FILE_BUILD_INFO_TMP = join-path $PATH_WORK_SERVER $DEF_FILE_INFO
if($DEBUG_MODE = 1) {
  write-output "[DEBUG] FILE_BUILD_INFO_TMP : $FILE_BUILD_INFO_TMP"
}

$PATH_S3_BUILD = "$AWS_S3_ADDR/" + "$GAME_BUILDZONE/" + "build/"
$PATH_S3_BUILD_CONF = "$AWS_S3_ADDR/" + "$GAME_BUILDZONE/" + "conf/"
$PATH_S3_SERVER = "$AWS_S3_ADDR/" + "$GAME_BUILDZONE/" + "build/" + "$GAME_BUILDTYPE/"
$PATH_S3_SERVER_CONF = "$AWS_S3_ADDR/" + "$GAME_BUILDZONE/" + "conf/" + "$GAME_BUILDTYPE/"
$FILE_S3_BUILD_INFO = "$AWS_S3_ADDR/" + "$GAME_BUILDZONE/" + "build/" + "$DEF_FILE_INFO"


if(!(Test-Path -Path $PATH_SERVER)){ mkdir $PATH_SERVER }
if(!(Test-Path -Path $PATH_BACKUP)){ mkdir $PATH_BACKUP }
if(!(Test-Path -Path $PATH_WORK)){ mkdir $PATH_WORK }
if(!(Test-Path -Path $PATH_WORK_SERVER)){ mkdir $PATH_WORK_SERVER }
if(!(Test-Path -Path $PATH_TMP)){ mkdir $PATH_TMP }

if(Test-Path $FILE_BUILD_INFO_TMP){
    remove-Item -force $FILE_BUILD_INFO_TMP
    write-output "[INFO] Completed for del to buildinfo.xml Temporary File"
}

if($DEBUG_MODE = 1) {
    write-output "[DEBUG] DEBUG MODE : On"
    if(Test-Path $FILE_BUILD_INFO){
      #remove-Item -force $FILE_BUILD_INFO
    }
    write-output "[DEBUG] Completed for del to buildinfo.xml File"
}

# 2. Download & Check for buildinfo.xml #
write-output "."
write-output "[INFO] BUILD Version Downloading & Checking Version..."
write-output "."


if(!(Test-Path $FILE_BUILD_INFO)){
    write-output "[INFO] File not found buildinfo.xml for Current BUILD"
    write-output "[INFO] Copy from Sample buildinfo.xml file"

    if(!(Test-Path -Path $PATH_SERVER)){ mkdir $PATH_SERVER }
    copy-item $FILE_BUILD_INFO_SAMPLE $FILE_BUILD_INFO
}

if(!(Test-Path $FILE_BUILD_INFO_TMP)){
    write-output "[INFO] Download from buildinfo.xml for NEW BUILD"

    aws s3 cp --region $AWS_S3_REGION $FILE_S3_BUILD_INFO $FILE_BUILD_INFO_TMP   > $null
    if($DEBUG_MODE = 1) {
      write-output "[DEBUG] aws s3 cp --region $AWS_S3_REGION $FILE_S3_BUILD_INFO $FILE_BUILD_INFO_TMP"
    }
}

[xml]$xml_cur_buildinfo = get-content $FILE_BUILD_INFO
$cur_ver = $xml_cur_buildinfo.File
[xml]$xml_new_buildinfo = get-content $FILE_BUILD_INFO_TMP
$new_ver = $xml_new_buildinfo.File

write-output "[INFO] +BUILD (Current Ver) = $cur_ver"
write-output "[INFO] +BUILD (New Ver) = $new_ver"


# -le means <=
if ($cur_ver -ne $new_ver) {
    Write-output "[INFO] Deploy start"

	
	$PATH_WORK_UNZIP = join-path $PATH_WORK_SERVER $new_ver
	if(!(Test-Path -Path $PATH_WORK_UNZIP)){ mkdir $PATH_WORK_UNZIP }

	
    #Download (BUILD)
    write-output "."
    write-output "[INFO] Downloading Build..."
    write-output "."

    # "GameServer-2016110300".zip/.tar.gz/
    $FILENAME_BUILD = "$new_ver" + ".zip"
    if($DEBUG_MODE = 1) {
    write-output "[DEBUG] FILENAME_BUILD : $FILENAME_BUILD"
    }
	
    $FILE_BUILD_WORK = join-path $PATH_WORK $FILENAME_BUILD
	
    if($DEBUG_MODE = 1) {
    write-output "[DEBUG] FILE_BUILD_WORK : $FILE_BUILD_WORK"
    }
    $FILE_S3_BUILD = "$PATH_S3_SERVER" + "$FILENAME_BUILD"
    if($DEBUG_MODE = 1) {
    write-output "[DEBUG] FILE_S3_BUILD : $FILE_S3_BUILD"
    }

    #download
    write-output "[INFO] Download to $FILE_S3_BUILD"
    aws s3 cp --region $AWS_S3_REGION $FILE_S3_BUILD $PATH_WORK > $null
    if($DEBUG_MODE = 1) {
    write-output "[DEBUG] aws s3 cp --region $AWS_S3_REGION $FILE_S3_BUILD $PATH_WORK"
    }
    $exe_zip = "$PATH_SCRIPT\unzip.exe"
    #uncompress
    #$exe_zip -x $FILE_BUILD_WORK $PATH_WORK_SERVER
	Invoke-Expression "& `"$exe_zip`" -o $FILE_BUILD_WORK -d $PATH_WORK_UNZIP"

    #Download (CONF)
    write-output "."
    write-output "[INFO] Downloading Build Configuration..."
    write-output "."
  
	
	
    $TARGET_PATH = join-path $PATH_WORK_UNZIP "."
	xcopy /E /Y $PATH_CONF\. $TARGET_PATH
    #aws s3 cp --region $AWS_S3_REGION --recursive $PATH_S3_SERVER_CONF $TARGET_PATH > $null
    if($DEBUG_MODE = 1) {
    write-output "[DEBUG] aws s3 cp --region $AWS_S3_REGION --recursive $PATH_S3_SERVER_CONF $TARGET_PATH"
    }

    #Deploy
    write-output "."
    write-output "[INFO] Deploying build..."
    write-output "."

    copy-item -recurse $PATH_WORK_UNZIP $PATH_SERVER
    write-output "[INFO] deploy source complted"
    copy-item -force $FILE_BUILD_INFO_TMP $PATH_SERVER
    write-output "[INFO] deploy buildinfo.xml complted"

    #clean
    write-output "."
    write-output "[INFO] Cleaning Work Build..."
    write-output "."

    if(Test-Path $FILE_BUILD_INFO_TMP){
        remove-item -force $FILE_BUILD_INFO_TMP
    }
	
    if(Test-Path -Path $PATH_WORK_SERVER){
        remove-item -recurse -force -path $PATH_WORK_SERVER
    }

    $TIME_CUR = Get-Date -UFormat '%Y-%m-%d %H:%M:%S'
    write-output "[INFO] END TIME : $TIME_CUR"
    Write-output "[INFO] Deploy done."

} else {

    Write-output "[INFO] Current BUILD is same or high version. Deploy cancel!!"
    $TIME_CUR = Get-Date -UFormat '%Y-%m-%d %H:%M:%S'
    write-output "[INFO] END TIME : $TIME_CUR"
}

```

## Rundeck Job 설정하기

기본적으로 배포를 위한 rundeck 프로젝트는 설정되어 있는 상태를 가정합니다.

프로젝트의 화면에서 [Create Job] 을 클릭하여, 신규 Job 을 생성합니다.

![1538052845860](/assets/rundeck-deplay-04.png)

**Job Name** : 작업의 간략한 이름을 설정 합니다. 

**Description** : 작업에 대한 처리 내용을 간략히 작성하여, 다른 작업자 분들이 해당 작업이 어떤 내용들을 수행하는지 작성해주시면 됩니다.

**Group** : 작업을을 한 그룹으로  묶어서 화면에 표시 되게 됩니다. 게임 관련 작업이면, GAME 등으로 식별이 가능한  이름을 작성하시면 됩니다.

![1538053123551](/assets/rundeck-deplay-05.png)

해당 노드들에게 실행할 커맨드를 입력을 하기 위해서 [Command] 버튼을 클릭합니다.

![1538055789901](/assets/rundeck-deplay-06.png)

**Command** : 실행할 Windows Command 명령을 입력 하시면 됩니다. 우리는 배포를 위해서 작성해 놓은 파워쉘 스크립트 파일명을 입력 하시면됩니다.

![1538055944987](/assets/rundeck-deplay-07.png)

**Node Filter** : 위의 명령을 수행할 노드들을 선택합니다. Rundeck 노드 설정에서 입력 한 값을 이용하여, 배포할 노드들을 선택 할수 있습니다.

**Matched Nodes** : 입력된 필터에 맞는 결과 노드들이 표시 되며, 작업을 수행시 이 노드들에 대해서 수행을 하게 됩니다.

![1538055989937](/assets/rundeck-deplay-08.png)



기본적인 입력항목은 작성이  되었습니다. 추가 적으로 작업의 성공 실패 여부를 Dooray 등으로 출력을 위해서는.

**Send Notification** [Yes] 를 클릭 후, **Http Notification** 을 선택하신 후에 항목들을 입력 하시면 됩니다.

## 마지막

이제 생성된 Jobs 을 이용해서 배포를 진행 하도록 하겠습니다.  위 방법으로 작업이 생성되면 아래의 그림과 같은 메뉴들을 볼수 있습니다. [GAME_DEPLOY] 를 클릭하여 배포를 진행 하도록 합니다.

![1538059516647](/assets/rundeck-deplay-09.png)

배포할 Nodes 를 선택한 후 [Run Job Now] 를 클릭합니다.

![1538059595584](/assets/rundeck-deplay-10.png)

정상적으로 실행된다면, 아래의 그림과 같이 수행되는 콘솔로그를 [Log Output] 탭에서 확인 할 수 있습니다.

![1538059753322](/assets/rundeck-deplay-11.png)