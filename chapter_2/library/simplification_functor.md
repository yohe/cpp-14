### *functorの利用簡単化*
---
#### 概要
一部のfunctorに対して、テンプレート引数を省略することが可能となった。
今までは、functorクラスに`operator()`の引数の型を示すテンプレート引数を指定する必要があった。
C++14では、functorクラスの`void`の特殊化定義が追加され、型の指定が不要となった。

以下のような定義が追加されている。
見ればわかるように、戻り値の型も型推論によって解決するため、既存の定義に比べ汎用化されている。
```c++
template <> struct greater<void> {
    template <class T, class U> constexpr auto operator()(T&& t, U&& u) const
        -> decltype(std::forward<T>(t) > std::forward<U>(u)) {
            return std::forward<T>(t) > std::forward<U>(u);
        }
};
```


以下に使用時のサンプルを示す。
```c++
#include <vector>
#include <algorithm>
#include <functional>

int main() {
    std::vector<int> v = {1,2,3,4,5};

    std::sort(v.begin(), v.end(), std::greater<int>());     //OK
    std::sort(v.begin(), v.end(), std::greater<>());        //OK C++14 only
    return 0;
}
```


特殊化定義が追加されたfunctor一覧を以下に示す。

* 四則演算 functor
 * std::plus
 * std::minus
 * std::multiplies
 * std::divides
 * std::modulus
 * std::negate

* 比較 functor
 * std::equal_to
 * std::not_equal_to
 * std::greater
 * std::less
 * std::greater_equal
 * std::less_equal

* 論理演算
 * std::logical_and
 * std::logical_or
 * std::logical_not
 * std::logical_not

* ビット演算
 * std::bit_and
 * std::bit_or
 * std::bit_xor
 * std::bit_not
