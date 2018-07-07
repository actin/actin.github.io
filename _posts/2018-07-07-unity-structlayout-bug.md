---
title: "Unity3D 2017 StructLayout 버그"
categories: program
tags: unity3d
toc: true
typora-root-url: ..
---



# Unity3D StructLayout 버그

개발중에 AOS mono 빌드로 테스트 중 발견된 내용입니다.

사용환경 : Unit3D 2017.3 0f3 .net 3.5 

아래와 같읕 통신을 위한 구조체에 값을 대입시 대입한 값이 달라지는 현상이 발생한다.

```c#
[StructLayout(LayoutKind.Sequential, Pack = 1)]
public struct DataInfo
{
    public short Val1;
    public short Val2;
    public short Val3;
    public byte Val3;
    public Int64 Val4;
    public int Val5;
    public byte Grade;
    public short Level;
    public byte AWake;
    public float Scale;
}
```



```c#
DataInfo info = new DataInfo();
info.Val1 = 1;
info.Val2 = 1;
info.Val3 = 1;
info.Val4 = 1;
info.Val5 = 1;
info.Grade = 1;
info.Level = 25;   // 이 값이 대입되면, Scale 값이 이상한 값으로 셋팅됨
info.AWake = 1;
info.Scale = 1f; // 이 값이 대입되면, Level = 0, AWake = 128 으로 값이 셋팅됨

```

이 현상은 mono 빌드에서 발생하며, 에디터와, IL2CPP 에서는 오류 없이 정상 동작한다.

mono의 낮은 버전으로 발생되는 현상으로  보이는데, 닷넷 버전을 4.6으로 설정후 테스트 해볼 예정이다.