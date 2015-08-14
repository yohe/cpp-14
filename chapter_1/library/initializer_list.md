### *initializer list*
---
#### 概要
クラスのコンストラクタが以下の条件を満たす場合、そのコンストラクタを *initializer-list constructor* と呼ぶ。

 * 最初の引数が`std::initializer_list<E>`、もしくはcv修飾を可能とするstd::initializer_list<E>の参照
 * その他の引数がない、もしくは残りすべてにデフォルト値が指定されている

initializer-list constructor を持つ場合、`{初期化リスト}`による初期化を使用すると他のコンストラクタより、initializer-list constructor が優先される。
ただし、initializer_list の要素が空である場合、デフォルトコンストラクタ呼び出しとして判断される。
`std::initializer_list<E>`を使用する場合、ヘッダファイル **initializer_list** を include する必要がある。


```c++11
#include <initializer_list>
#include <algorithm>
#include <cassert>

class InitList {
public:
    template <class E>
    InitList(std::initializer_list<E> lists) {}     //#1

    InitList(int a, int b) {}                       //#2

    InitList() {}                                   //#3
};


int main() {
    InitList a {1, 2, 3, 4, 5};     //called #1
    InitList b {1, 2};              //called #1
    InitList c (1, 2);              //called #2
    InitList d { };                 //called #3
    return 0;
}
```
