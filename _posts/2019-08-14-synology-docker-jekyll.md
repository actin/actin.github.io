---
layout: post
title : "Synology nas에서 docker를 이용한 jekyll 사용하기"
categories : devlog
tags : jekyll docker synology
toc : true
typora-root-url : ..
---



## Synology Jekyll Docker

github.io 도 있지만 개인 보유하고 있는 Nas 를 이용해서 개인 블러그를 운영을 위해서 작성합니다.
하루 종일 이것 저것 삽질하면서 잘 안되 포기 할까도 생각했지만 오기 발동하여 결국 성공!!
Jekyll Docker 의 문서에 아주 잘 나와 있는데 <span style="color:blue">Synology</span> 에서 적용 한다고 삽질을....

<https://github.com/envygeeks/jekyll-docker/blob/master/README.md>

간략히 내용을 정리 해보면, jekyll_home 생성 후  bundle 파일을 저장하기 위한 커스텀 폴더 생성  jekyll bundle  update 를 실행하여 필요한 bundle 파일을 업데이트 후 -w 옵션을 이용한 변경 감지를 통해서 자동 정적 페이지 생성을 한다.  입니다.

`Docker` 저장소에서 `jekyll` 검색한후 `jekyll/jekyll` 이미지를 다운로드 합니다.![](/assets/jekyll-1.png)



`jekyll`  의 원본이 위치한 경로의 접근 권한을 `everyone` 으로 읽기쓰기가 가능 하도록 먼저 설정합니다. 실행권한을 최고 권한으로 실행했지만 결국은 권한이 없다고 에러가.. 꼭! `everyone` 으로 읽기 쓰기를 설정하세요.

저는 jekyll_home 공유 폴더를 생성 후 이미 theme 를 생성한 폴더와 파일을 준비 하였습니다. 이 부분은 여기서 다루지 않을 예정입니다. 

![1565786005518](/assets/jekyll-6.png)





## Dependencies

jekyll Docker `Gemfile` 의 리스트에서 의존성 파일을 설치 하게 됩니다.

### Updatding & Caching

`jekyll_home/vendor/bundle` 폴더는 생성해주세요.  `Gemfile` 을 제공한다면, 아래와 같이 실행 합니다.



```
docker run --rm \
  --volume="/volume1/jekyll_home:/srv/jekyll" \
  --volume="/volume1/jekyll_home/vendor/bundle:/usr/local/bundle" \
  -it jekyll/jekyll \
  bundle update
```

필요한 파일을 자동으로 다운로드 하여, vendor/bundle 폴더에 필요한 파일을 자동으로 다운을 시작합니다. 이부분을 저는 잘 이해를 못해서 하루 삽질했네요.

![1565786178287](/assets/jekyll-7.png)

`/jekyll_home/vendor/bundle` 폴더에는 아래와 같이 폴더와 파일이 생성된것 을 확인 할 수 있습니다.

![1565786244229](/assets/jekyll-8.png)

## Build(생성하기)

이제 정적 페이지를 생성 하기 위해서는 bundle 에 캐싱한 내용을 이용하여 아래의 명령을 실행하면,  `jekyll_home/_site` 폴더가 생성되고 정적 파일들이 생성되는 것을 확인 할 수 있습니다.

```
docker run --rm \
  --volume="/volume1/jekyll_home:/srv/jekyll" \
  --volume="/volume1/jekyll_home/vendor/bundle:/usr/local/bundle" \
  -it jekyll/jekyll \
  jekyll build
```

`_post` 폴더의 변경된 내용을 자동으로 감지후 페이지 생성을 위해서는 `synology` 에서 `Docker` 이미지를 실행하여 자동으로 생성 하도록 하면 됩니다. 



## Synology Docker 를 이용하여 자동 변경 감지 후 자동 생성하자

이미지를 클릭 후 실행 버튼을 클릭하여 컨테이너를 생성 하도록 합니다.

![1565784898282](/assets/jekyll-2.png)

컨테이너 이름은 적당한 이름을 작성하고 고급 설정을 클릭하여 , 캐싱한 bundle 경로와 jekyll 문서가 있는 루트 경로를 볼륨으로 추가합니다.

![1565785185185](/assets/jekyll-3.png)

볼륨 추가

![1565785216697](/assets/jekyll-4.png)

환경 설정 탭에서 실행명령 란에 `jekyll build -w`  을 입력후 적용 버튼 누릅니다.

![1565785292831](/assets/jekyll-5.png)

모든게 완료 되었다면  `jekyll_home/_site` 폴더가 생성된 후 정적페이지를 위한 파일들이 자동 생성 됨을 확인 할 수 있습니다.





