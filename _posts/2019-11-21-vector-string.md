---
layout: post
title : "C++ std::vector to std::string"
subtitle: "converting vector to std::string "
categories : devlog
tags : c++
typora-root-url : ..
---

가끔 vector의 내용을 `콤마` 로 구분된 출력을 하고 싶을 때가 종종 발생한다. 

## std::vector to std::string


```c++
#include <vector>
#include <string>
#include <algorithm>
#include <sstream>
#include <iterator>
#include <iostream>


int main()
{
  std::vector<int> vec;
  vec.push_back(1);
  vec.push_back(4);
  vec.push_back(7);
  vec.push_back(4);
  vec.push_back(9);
  vec.push_back(7);

  std::ostringstream oss;

  if (!vec.empty())
  {
    
    std::copy(vec.begin(), vec.end()-1,
        std::ostream_iterator<int>(oss, ","));
    
    oss << vec.back();
  }

  std::cout << oss.str() << std::endl;
}
```


## std::vector to std::wstring

```c++
#include <vector>
#include <string>
#include <algorithm>
#include <sstream>
#include <iterator>
#include <iostream>

int main()
{
  std::vector<int> vec;
  
  vec.push_back(1);
  vec.push_back(4);
  vec.push_back(7);
  vec.push_back(4);
  vec.push_back(9);
  vec.push_back(7);

  std::wostringstream oss;

  if (!vec.empty())
  {
    // Convert all but the last element to avoid a trailing ","
    std::copy(vec.begin(), vec.end()-1,
        std::ostream_iterator<int, wchar_t, std::char_traits<wchar_t> >(oss, L","));

    // Now add the last element with no delimiter
    oss << vec.back();
  }

  std::wcout << oss.str() << std::endl;
}
```


[http://cpp.sh/45sy32](http://cpp.sh/45sy32)