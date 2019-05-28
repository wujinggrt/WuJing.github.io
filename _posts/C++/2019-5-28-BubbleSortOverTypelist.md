---
layout: post
title: 使用 typelist 进行编译期冒泡排序
topic: C++
category: C++
---

背景：刚刚看完了《Modern C++ Design》第一、二、三、七章，选择实现一个小玩具来练练手，巩固学习到的知识。

复杂度是 O(n^2)。

数据组织方式主要实现依赖于第三章的 `typelist` ，把所有的数据都映射成类型，从而能够在编译期通过对特例化和偏特例化的匹配，完成逻辑判断的功能。

排序部分是通过遍历 `typelist` ，比较两个相邻结点，决定交换与否。一共进行 `n` 次这样的遍历，完成冒泡排序，得到有序结果。

```C++
#include <iostream>
#include <type_traits>

namespace wj {

struct null_type {};

template<typename Head, typename Tail>
struct typelist {
    using head = Head;
    using tail = Tail;
};

template<typename T, T... args>
struct typelist_builder;

template<typename T, T... args>
using typelist_builder_t = typename typelist_builder<T, args...>::type;

template<typename T, T v>
struct typelist_builder<T, v> {
    using type = typelist<std::integral_constant<T, v>, null_type>;
};

template<typename T, T v, T... args>
struct typelist_builder<T, v, args...> {
private:
    using rest_typelist = typelist_builder_t<T, args...>;
public:
    using type = typelist<std::integral_constant<T, v>, rest_typelist>;
};

template<typename T> struct swap_with_next;

template<typename T>
using swap_with_next_t = typename swap_with_next<T>::type;

template<typename Head, typename Tail>
struct swap_with_next<typelist<Head, Tail>> {
    using type = typelist<typename Tail::head, typelist<Head, typename Tail::tail>>;
};

template<typename Head>
struct swap_with_next<typelist<Head, null_type>> {
    using type = typelist<Head, null_type>;
};

// 每个 swap_helper 遍历整个 list，然后比较遍历到位置和下一个位置，
// 视情况交换两个节点。
template<typename TList> struct swap_helper;

template<typename TList>
using swap_helper_t = typename swap_helper<TList>::type;

template<>
struct swap_helper<null_type> {
    using type = null_type;
};

template<typename Head>
struct swap_helper<typelist<Head, null_type>> {
    using type = typelist<Head, null_type>;
};

template<typename Head, typename Tail>
struct swap_helper<typelist<Head, Tail>> {
private:
    using ordered = 
        std::conditional_t<(Head::value < Tail::head::value),
                           typelist<Head, Tail>,
                           swap_with_next_t<typelist<Head, Tail>>>;
public:
    using type = 
        typelist<typename ordered::head, swap_helper_t<typename ordered::tail>>;
};

template<typename Traverse, typename TList>
struct bubble_sort_helper;

template<typename Traverse, typename TList>
using bubble_sort_helper_t = typename bubble_sort_helper<Traverse, TList>::type;

// 最后一个节点。
template<typename Head, typename TList>
struct bubble_sort_helper<typelist<Head, null_type>, TList> {
    using type = swap_helper_t<TList>;
};

template<typename Head, typename Tail, typename TList>
struct bubble_sort_helper<typelist<Head, Tail>, TList> {
private:
    using swapped = bubble_sort_helper_t<Tail, TList>;
public:
    using type = swap_helper_t<swapped>;
};

template<typename TList> struct bubble_sort;

template<typename TList>
using bubble_sort_t = typename bubble_sort<TList>::type;

template<typename Head>
struct bubble_sort<typelist<Head, null_type>> {
    using type = typelist<Head, null_type>;
};

template<typename Head, typename Tail>
struct bubble_sort<typelist<Head, Tail>> {
    using type = 
        bubble_sort_helper_t<typelist<Head, Tail>, typelist<Head, Tail>>;
};

template<typename T> struct printer;

template<typename Head>
struct printer<typelist<Head, null_type>> {
    static void print() {
        std::cout << Head::value << std::endl;
    }
};

template<typename Head, typename Tail>
struct printer<typelist<Head, Tail>> {
    static void print() {
        std::cout << Head::value << " ";
        printer<Tail>::print();
    }
};

}

using namespace wj;

int main() {
    using unsorted_t = 
        typelist_builder_t<decltype(61), 61, 17, 29, 22, 
                           34, 60, 72, 21, 50, 1, 62, 0>;
    std::cout << "Before bubble_sort: ";
    printer<unsorted_t>::print();
    using sorted_t = bubble_sort_t<unsorted_t>;
    std::cout << "After bubble_sort: ";
    printer<sorted_t>::print();
}
```

结果：

    || Before bubble_sort: 61 17 29 22 34 60 72 21 50 1 62 0
    || After bubble_sort: 0 1 17 21 22 29 34 50 60 61 62 72

读了这几章后，明显感觉到元编程需要函数式编程的底子，思考方式和 `OOP` 范式不同，继续学习的话，还需要看 `Haskell` 的书和《C++ Templates》才能够继续玩弄。不过目前也不打算在继续学习下去，至少要等到一段时间以后才会有可能重新回来看，时间还应该放在其他比如 `Linux` 等知识上，弄清主次。其次， `typelist` 也应该能够支撑大部分应用情况了，也算是停留在走火入魔的前期。