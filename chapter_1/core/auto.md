### *auto*
---
#### 概要

C++03では自動変数(auto変数)を宣言する際の記憶クラス指定子(staticなど)であったが、通常は自動変数とみなされる(ブロックスコープの変数はデフォルトで指定される)ためC++11において廃止となり、**型推論を示す型**として再定義された。

キーワード"auto"を変数を宣言する際の型として指定すると、autoの型は初期化式( = XXX; )により決定される。
従って、autoを指定した変数宣言には、常に**初期化式が必須**となる。
また、型推論の結果の型は、初期化式をテンプレート関数に渡した際に使用される型と一致する。

autoは型指定子であるため、他の型指定子と同様にcv修飾子(const/volatile)、参照修飾子(&/&&)も付加可能となる。

```c++
class X {
public:
    X() {}
    X(const X& lhs) {
    }
};

X f();
X& f_ref();

int main(int argc, char** argv) {
    auto i = 1;             // int
    auto f = 1.0f;          // float
    auto d = 10.0;          // double

    auto v = f();           // X
    auto & v_ref = f_ref(); // X &
}
```

また、キーワード"auto"は、関数の戻り値の型推論にも使用することができる。但しreturn文からの型推論ではなく、
Trailing-return-typeと呼ばれる後置形の型指定の式の型推論から決定する。

Trailing-return-type指定を使用する場合は、以下のような構文となる。

`auto 関数名(引数リスト) -> 式 { body }`

Trailing-return-typeが追加された経緯として、以前は戻り値の型を導出するために、引数リストの変数を参照することが出来なかった。

```c++
auto func() -> int {
    return 0;
}

auto func() -> double {
    return 0;
}
```

autoで宣言された変数と同じ型の変数を再度作成したいと言った場合もある。この場合、decltype指定子というものを使用する。
decltype指定子とは、decltype(式)という記述で**型**を表すことができ、ここで表される型はdecltypeに指定した式の型である。
また、decltypeで指定した式は評価されない(ランタイム中に実行されない)。


```c++
double f();

auto i=0;           //i is int
decltype(i++) j;    //j is int
decltype(f()) d;    //d is double

std::cout << i << std::endl; // 0
```

また、このdecltypeは型の決め方に特殊なルールが設けられている。
以下にそのルールをまとめる。decltype(e)の式eについて、以下の評価順で型判定が行われる。

 1. 式e が括弧で囲まれず、名前(変数名, function, etc...)もしくはクラスメンバへのアクセス(x.a or x->a)である場合、指定した名前の型である。但し、名前が存在しない、関数名が複数存在する場合はill-formed
 1. 式e が[rvalue参照](chapter_1/core/r_value_ref.md)として判定される場合、T&& (Tは式eの型)
 1. 式e が括弧で囲まれた変数名である場合、T& (Tは式eの型)
 1. 記のどれにも該当しない場合、式eの型

```c++
int&& xvalue();

struct X {
    double a;
};

int main() {
    int a;
    X x;
    const X* xp = new X();
    decltype(a)     v1;     //type is int
    decltype(x.a)   v2;     //type is double
    decltype(xp->a) v2;     //type is double
}
```

前述のTrailing-return-typeは、このdecltypeを用いることで真価を発揮する。以下に幾つかの例を示す。
Trailing-return-type版と、通常版をそれぞれ記載する。

```
template <class T, class U>
decltype((*(T*)0) + (*(U*)0)) add(T t, U, u) { return t + u; }

template <class T, class U>
auto add(T t, U, u) -> decltype(t + u) { return t + u; }


int (*(*bar())())() { return 0; }     // intを戻り値とする関数ポインタを戻り値とする関数ポインタを返却する関数bar

auto bar() -> auto (*)() -> int (*)() { return 0; }
```

さて、autoによる変数宣言を行うと(特に初期化式に関数呼び出しが含まれる場合)、変数の型が人間には判断しづらくなるというデメリットがある。
だが、最近の開発環境では関数宣言へのジャンプ機能などがあるため大きな問題になることは少ないと思われる。それよりも、autoによる型のロバスト性獲得であったり、 コンパイラによる最適な型の選択によるメリットの方が大きい場合もあると考える。

例えば、以下のようなケースを考える。

```c++
#include <map>
#include <string>
#include <iostream>

std::map<std::string, int> m;

int main() {
    m.insert(std::make_pair(std::string("abc"), 1);

    for (const std::pair<std::string, int>& p : m) {
        std::cout << p.first << std::endl;
    }
}
```
上記のコードには意図しない動作が含まれている。for文の要素を参照で受け取っているが、
実は型が異なることによってコピーが発生するため、参照として受け取れていない。
実際の型は、`std::pair<const std::string, int>` である。
以下のコードを実行することで確認可能である。

```c++
#include <map>
#include <iostream>

class X {
public:
    X(int i) : i_(i) {}
    X() = default;
    ~X() = default;


    X(const X& x) {
        std::cout << "copy construct" << std::endl;
    }
    X& operator=(const X& x) = default;

    X(X&& x) = default;
    X& operator=(X&& x) = default;

    int i_;
};

bool operator<(const X& r, const X& l) {
    return r.i_ < l.i_;
}

std::map<X, int> m;

int main() {
    m.insert(std::make_pair(X(1), 1));
    m.insert(std::make_pair(X(2), 2));

    std::cout << "----------------------" << std::endl;
    for( const std::pair<X, int>& p : m) {
        std::cout << p.second << std::endl;
    }

    std::cout << "----------------------" << std::endl;
    for( const std::pair<const X, int>& p : m) {
        std::cout << p.second << std::endl;
    }
    return 0;
}
```

```
----------------------
copy construct
1
copy construct
2
----------------------
1
2
```

上記のような間違いは起こりやすい。このような間違いは、autoを使用することで無くすことができる。
autoを使用しすぎると保守性が低下する恐れがあるが、上記のような型が自明な箇所では積極的に使用していくのが良いと思う。

```c++
#include <map>
#include <string>
#include <iostream>

std::map<std::string, int> m;

int main() {
    m.insert(std::make_pair(std::string("abc"), 1);

    for (const auto& p : m) {
        std::cout << p.first << std::endl;
    }
}
```

