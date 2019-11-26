---
layout: post
title : "C++ std::vector to std::string"
subtitle: "converting"
categories : devlog
tags : c++
toc : true
typora-root-url : ..
---

##std::vector to std::string

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

##std::vector to std::wstring

```
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



http://cpp.sh/45sy32