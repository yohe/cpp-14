### *initializer list*
---
#### 概要
クラスのコンストラクタが、以下の条件を満たす場合、そのコンストラクタを *initializer-list constructor* と呼ぶ。
 * 最初の引数が`std::initializer_list<E>`、もしくはcv修飾を可能とするstd::initializer\_list<E>の参照
 * その他の引数がない、もしくは残りすべてにデフォルト値が指定されている

initializer-list constructor を持つ場合、`{初期化リスト}`による初期化を使用すると他のコンストラクタより、initializer-list constructor が優先される。
`std::initializer\_list`を使用する場合、**initializer_list**を include する必要がある。



```c++11
#include <initializer_list>

class X {
public:
    template <class E>
    X(std::initializer_list<E> lists) {
    }
};
```
