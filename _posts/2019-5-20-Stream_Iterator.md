---
layout: post
title: Stream Iterator
---

# C++ 中流迭代器的小节

在流的类中，内部永远只能够存在一个某种意义上的“指针”，指向当前`stream`中的特定位置，就好比数组的位置一样。但是有一个特点，不能够**回退**，不能够有`--`的操作。

第二个特性，流迭代器只能够按照`++`前进，不能够随机访问（`stream_iterator + n`)的操作。

基于上面描述，得到一个结论：就算在当前迭代器(例子中的`iter`)复制当前的流迭代器(新的临时迭代器`next(iter, 2)`)之后,复制版本迭代器执行增加操作，得到新的位置后，原版本的迭代器就算执行`iter++`操作也不会获得下一个位置，而是那个拷贝迭代器的下一个位置。中心思想就是不可能从流中获得不同位置的迭代器，除非是`copy`之后，不进行新的操作，也就是初始的状态。

```C++
#include <string>
#include <iostream>
#include <iterator>
#include <algorithm>
#include <vector>
#include <sstream>
using namespace std;

int main() {
    stringstream ss;
    ss << "First " << "second " << "third " << "fourth ";

    auto citer = ostream_iterator<string>(cout);
    *citer++ = "cout: third";
    *citer++ = " fourth\n";

    auto oiter = ostream_iterator<string>(ss);
    *oiter++ = " fifth";
    *oiter++ = " 123";
    string stmp;
    // 内部有一个自带的迭代器，与上一个 oiter 相似，指向相同的地方
    // 所以下一个 iter 已经指向了 second 了。
    ss >> stmp;
    *citer++ = stmp;
    // 如果没有让迭代器下一，那么就会append到上一个末尾。
    *citer = "   padding"; X
    cout << "\n";
    auto iter = istream_iterator<string>(ss);
    cout << *iter++ << "\n";
    cout << *next(iter， 2) << "\n";
    cout << *iter++ << "\n";
    // 这儿已经跳到了下一个了，而不是fourth
    cout << *iter++ << "\n";
    *citer++ = ss.str();
    cout << "\n";
    *citer++ = ss.str();
    cout << "\n";
}
```

输出如下:

```C++
// output
// cout: third fourth
// First   padding
// second
// fifth
// third
// 123
// First second third fourth  fifth 123
// First second third fourth  fifth 123
// 
// 
// [Finished in 1 seconds]
```