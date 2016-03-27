### *ratio*
----
#### 概要
コンパイル時の有理数表現機能を提供する。
有理数は、テンプレートクラスである*ratio*に、分子と分母をテンプレート引数として渡すことで構築する。
分母が0, もしくは絶対値が*intmax_t*の型で表現できない場合、ill-formedとなる。
また標準では、SI単位の接頭辞(kiro,giga,milli,etc...)が定義されている。ヨタからヨクトまでが標準で規定されているが、
ヨタやゼタなどで、intmax_tの型で表現できない大きさについては定義されない。

本節で解説する機能を使用するには、ヘッダファイル **ratio**をインクルードする。

またratioには、四則演算機能と、比較機能が提供されている。

##### 四則演算
四則演算機能は、ratioクラスの*型*を引数にとり、ratioクラスを定義するエイリアステンプレートとして定義されている。

| 演算 | エイリアステンプレート名 |
| -- | -- | -- |
| 加算 | ratio_add |
| 減算 | ratio_subtract |
| 乗算 | ratio_multiply |
| 割算 | ratio_divide |

以下に使用方法を示す。

```c++
#include <ratio>
#include <iostream>

int main() {
    std::ratio<1, 5> x;
    std::ratio<2, 3> y;
    std::cout << std::ratio_add<decltype(x), decltype(y)>::num << "/"               //OK decltypeを使用できる
              << std::ratio_add<decltype(x), decltype(y)>::den << std::endl;

    std::cout << std::ratio_subtract<std::ratio<1, 2>, std::ratio<3,4>>::num        //OK 直接型を指定できる
              << "/"
              << std::ratio_subtract<std::ratio<1, 2>, std::ratio<3,4>>::den
              << std::endl;

    auto mul = std::ratio_multiply< decltype(x), decltype(y) >();                   //OK 結果を変数に格納できる
    std::cout << mul.num << "/"
              << mul.den << std::endl;
    
    return 0;
}
```

実行結果は以下のようになる。
```
13/15
-1/4
2/15
```

##### 比較機能
比較機能は、指定する２つの有理数が等しいかどうか、大小比較をコンパイル時に評価することができる。
比較の結果は、クラスの型として表現される。この型は、`integral_constant<bool, result>`を継承して表現され、staticメンバ変数valueに比較結果の真偽が設定される。

比較操作の一覧を示す。

| 比較 | 比較クラス名 |
| -- | -- | -- |
| 等号 | ratio_equal |
| 不等号 | ratio_not_equal |
| 未満 | ratio_less |
| 以下 | ratio_less_equal |
| 超過 | ratio_greater |
| 以上 | ratio_greater_equal |

```c++
#include <iostream>
#include <ratio>

int main(int argc, char const* argv[])
{
    std::ratio<1, 5> x;
    std::ratio<2, 3> y;

    if(std::ratio_equal<decltype(x), decltype(y)>::value) {
        std::cout << "ERR" << std::endl;
    } else {
        std::cout << "OK" << std::endl;
    }

    if(std::ratio_not_equal<decltype(x), decltype(y)>::value) {
        std::cout << "OK" << std::endl;
    } else {
        std::cout << "ERR" << std::endl;
    }

    if(std::ratio_less<decltype(x), decltype(y)>::value) {
        std::cout << "OK" << std::endl;
    } else {
        std::cout << "ERR" << std::endl;
    }

    return 0;
}
```

実行結果は以下のようになる。
```
OK
OK
OK
```
