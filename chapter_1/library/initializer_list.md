### *initializer list*
---
#### 概要
C++11で、統一初期化構文が追加され、"{初期化リスト}"による初期化が可能となった。以下の構文見た時、それぞれどのような動作となることを意図するか。

```c++
int array[] {1, 2, 3, 4, 5};
std::vector<int> v {1, 2, 3, 4, 5}
```

上記の構文は、どちらも要素数5で、1..5の配列となることを意図したいだろうと思われる。しかし、C++03のvectorには上記を実現するコンストラクタは存在しない。
そこでC++11では、クラスを"{初期化リスト}"による初期化の動作を変更する仕組みが導入されている。これが、initializer_listである。

クラスのコンストラクタが以下の条件を満たす場合、そのコンストラクタを *initializer-list constructor* と呼ぶ。

 * 最初の引数が`std::initializer_list<E>`、もしくはcv修飾を可能とするstd::initializer_list<E>の参照
 * その他の引数がない、もしくは残りすべてにデフォルト値が指定されている

initializer-list constructor を持つ場合、`{初期化リスト}`による初期化を使用すると他のコンストラクタより、initializer-list constructor が優先される。
ただし、initializer_list の要素が空である場合、デフォルトコンストラクタ呼び出しとして判断される。
`std::initializer_list<E>`を使用する場合、ヘッダファイル **initializer_list** を include する。


```c++
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
