---
layout: post
title: "ta-lib 컴파일하기"
categories: devlog
tags: ta-lib
toc: true
---

TA-lib 는 주식 분석에 필요한 다양한 기능을 포함하고 있다.
`C` 언어로 작성되었고 python 에서도 사용 할수 있다.

최근 우분투 환경에서 `Python` 을 이용하여 재미로 코인 관련 프로그램을 작성하면서 `ta-lib` 가 필요하게 되어 컴파일 하는 방법을 간략히 정리 한다.
기본적인 설치 방법은 [python 홈페이지](https://pypi.org/project/TA-Lib/) 를 참고 하면 다양한 방법으로 설치 가능하다.
하지만 설치시 따로 컴파일을 다시 해줘야 한다는 사실을 알게 되었다... 


# 우분투 컴파일 하는법
```bash
wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz

tar zxvf ta-lib-0.4.0-src.tar.gz 
cd ta-lib
./configure
make
sudo make install

pip3 install ta-lib
```

# arm64 컴파일 하는법
여러 사이트를 검색 하며 방법을 확인 했지만, 설정시 `--build=aarch64-unknown-linux-gnu` 추가 만으로 쉽게 해결이 되었다는

```bash
wget http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz

tar zxvf ta-lib-0.4.0-src.tar.gz 
cd ta-lib
./configure --build=aarch64-unknown-linux-gnu --prefix=/usr
make
sudo make install

pip3 install ta-lib
```