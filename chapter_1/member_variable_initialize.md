### *メンバ変数のデフォルト初期化*
---
#### 概要
C++03では、メンバ変数の初期化はコンストラクタの定義時に行われなくてはならなかった。また、複数のコンストラクタを定義する場合に、各コンストラクタで同じ値で初期化するようなメンバ変数は、毎度、初期化文を記述する必要があった。これは、面倒であるし初期化漏れによるバグを引き起こす要因となる。
そこで、C++11では、メンバ変数にデフォルト値を持たせることが可能となった。このデフォルト値は、コンストラクタで対象のメンバ変数を初期化しない場合に適用される。

```
#include <iostream>
#include <string>

class X {
public:
    X(std::string x) : value(x) {
        std::cout << x << std::endl;
    }

    std::string value;
};


class Y {
public:
    Y() {}
    Y(int) : x("Y constructor") {}

    X x = X("default");
};


int main(int argc, char const* argv[])
{

    Y a;        // xは"default"で初期化はされる
    Y b(1);     // xは一度も"default"での初期化はされない
    return 0;
}

```
これの実行結果は次のようになる。

```
default
Y constructor
```

