---
layout : post
title : pandoc 으로 markdown 을 docx 만들기
tags : tip
typora-root-url: ..
---

markdown 형식은 문서 정리에 최적화 되었다고 생각합니다. 마침 회사에서 문서화 작업을 진행하다가 좀 더 편하게 문서를 만들어서 MS 워드 파일로 만드는 방법은 없는지 찾다가 삽질을 기록합니다.



## Pandoc 이란?

다양한 포맷으로 markdown 뿐만 아니라 docx, html 등의 문서로 변환하는 유틸입니다. 
자세한 설명은 아래 사이트를 참고 하시면 엄청난 기능을 확인 할 수 있습니다.

[pandoc]: https://pandoc.org



## Ms word docx 생성을 위한 Custom Style 준비

docx 파일을 생성 후, 스타일 수정을 해도 되지만 pandoc 에서 지정한 스타일을 미리 수정한다면 빠르게 문서를 생성 할 수 있습니다.

```
pandoc --print-default-data-file reference.docx > custom-reference.docx
```



## Markdown CodeBlock 을 위한 스타일 지정하기

이 부분에서 많은 삽질이 있었습니다. 코드 블록(Code Block)에 테두리를 만들고 싶었는데 찾기가 정말 어려웠습니다.

하루 정도 검색 후 찾은 해답은

> The preformatted source is now formatted using the hidden linked style "Source Code". This style is NOT displayed in the styles preview pane by default, you can add it by configuring the styles preview pane by clicking the tiny square-with-arrow in the bottom right of the styles preview panel.

결론은 코드 블록에 대한 스타일이 숨겨져 있기 때문에 만들어라. 

그래서 우리가 준비한 custom-reference.docx 파일에서 스타일 만들기를 클릭하여 `Source Code` 라는 스타일을 만들었습니다.



## Docx 파일 생성하기



```
pandoc main.md -f markdown --resource-path=d:\doc \
--self-contained --data-dir=d:\doc\ref \
--reference-doc=custom-reference.docx -s -t docx \
-o d:\sample1.docx
```



### Options


* --resource-path
  마크다운 문서 작성시 사용되는 이미지가 있는 상위 폴더를 지정합니다.

* --self-contained
  docx 파일은 이미지와 같은 리소스를 자체 내장 하도록 설정하는 옵션입니다.

* --data-dir
  custom-referenct.docx 파일이 있는 위치를 지정합니다.

* --reference-doc
  스타일 지정을 위한 docx 파일명을 입력합니다. 위에서 생성한 `custom-referenct.docx` 입니다.

