### *タプル*
----
#### 概要
任意の型の引数を、任意の個数を指定して構築する事が出来る。タプルはstd::pairを任意の個数に一般化したものと考えればよい。 
タプルは指定した型のリストを含めてタプルの型として扱われるため、渡した引数の型の並びや個数が異なるならば異なる型として扱われる。
タプルは、ヘッダファイル**tuple**をインクルードすることで使用出来るようになる。

タプルから値を取り出すには、`std::get`, `std::tie`を用いる。
std::getは、指定した位置の値を取得する事ができ、std::tieはタプルの各位置を変数にマッピングでき、そこに取得対象のタプルを代入することで取得できる。
またstd::tieを用いる時、取得不要な位置に`std::ignore`を指定することでその位置の要素を無視することが出来る。

```c++
#include <iostream>
#include <tuple>

int main() {
    std::tuple<> t0();
    std::tuple<int, char> t2(1, 'a');
    std::tuple<int, char, double> t3(1, 'a', 1.1);

    std::cout << std::get<0>(t0) << std::endl;          //error 0番目の要素がない

    std::cout << std::get<0>(t2) << std::endl;          //OK 1
    std::cout << std::get<1>(t2) << std::endl;          //OK 'a'
    std::cout << std::get<2>(t2) << std::endl;          //error 2番目の要素がない

    std::cout << std::get<2>(t3) << std::endl;          //OK 1.1

    int i;
    double d;

    std::tie(i, std::ignore, d) = t3;                   //OK int->i, char->ignore, double->d
    std::cout << i << std::endl;                        //OK 1
    std::cout << d << std::endl;                        //OK 1.1

    return 0;
}
```

タプルを構築する便利関数std::make_tupleも提供されている。これは、std::make_pairと同じように使用出来る。

```c++
#include <tuple>

int main() {
    auto t = std::make_tuple(0, 'a', 0.0);
    return 0;
}
```

タプルには比較演算子が定義されており、これはタプルの先頭要素から同じ位置の要素同士を比較してくれる。
これを用いることで、ユーザ定義クラスの比較演算子を簡単に定義できる。
例えば`operator==()`であれば、全ての要素同士が一致している場合にtrueを返す。
`operator<()`であれば、要素同士の大小比較を行い、全ての要素が `a < b` を満たす場合に true を返す。

```c++
#include <iostream>
#include <string>
#include <tuple>

struct X {
   int i;
   char c;
   std::string str;
};

bool operator<(const X& lhs, const X& rhs) {
    return std::tie(lhs.i, lhs.c, lhs.str) < std::tie(rhs.i, rhs.c, rhs.str);
}
bool operator>=(const X& lhs, const X& rhs) {
    return !(lhs < rhs);
}
bool operator>(const X& lhs, const X& rhs) {
    return rhs < lhs;
}
bool operator<=(const X& lhs, const X& rhs) {
    return !(rhs < lhs);
}
bool operator==(const X& lhs, const X& rhs) {
    return std::tie(lhs.i, lhs.c, lhs.str) == std::tie(rhs.i, rhs.c, rhs.str);
}
bool operator!=(const X& lhs, const X& rhs) {
    return !(lhs == rhs);
}

int main() {
    X lhs {0, 'a', "abc"}
    X rhs1 {1, 'a', "abc"}
    X rhs2 {0, 'b', "abc"}
    X rhs3 {0, 'a', "efg"}

    if (lhs < rhs1) {
        std::cout << "lhs < rhs1" << std::endl;
    }
    if (lhs < rhs2) {
        std::cout << "lhs < rhs2" << std::endl;
    }
    if (lhs < rhs3) {
        std::cout << "lhs < rhs3" << std::endl;
    }
    if (lhs == lhs) {
        std::cout << "lhs == lhs" << std::endl;
    }
    return 0;
}
```
```
lhs < rhs1
lhs < rhs2
lhs < rhs3
lhs == lhs
```

