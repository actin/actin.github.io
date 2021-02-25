---
layout: post
title : "MacOS vnc command line"
subtitle: "Python boto3 copy, upload, invalidation"
categories : tip
tags : maxos vnc
typora-root-url : ..
---







ssh 접속으로 vnc 모드를 활성화할 필요가 있습니다.

## 모든 유저를 대상

~~~bash
sudo /System/Library/CoreServices/RemoteManagement/ARDAgent.app/Contents/Resources/kickstart \
  -activate -configure -access -on \
  -configure -allowAccessFor -allUsers \
  -configure -restart -agent -privs -all
~~~

