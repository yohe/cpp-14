### *constexpr関数の制限緩和*
---
#### 概要
C++11までのconstexpr関数、及びconstexprコンストラクタの関数本体の記述には厳しい制限があった。このため、constexprの関数で複雑な処理を記述する事が難しかった。また、constexpr指定のメンバ関数は暗黙的にconst指定されたものとして扱われるため、同じシグネチャを持つ関数でconst版と非const版のconstexpr関数を混在させる事が出来なかった。
C++では、戻り値の型のみが異なる関数のオーバーロードは許可されていない。


C++14では、これらの制限、制約が緩和されたためconstexpr関数の使用が大きく改善されている。
ここでは、constexpr関数、及びconstexprコンストラクタの本体に記述できる内容について示す。

##### constexpr関数
C++14のconstexpr関数本体には、以下で示す構文以外が記述可能である。
 * asm定義 (インラインアセンブラ)
 * goto文
 * tryブロック
 * 非リテラル型、もしくはstatic or thread_local指定の変数の定義
 * 初期化が実行されない変数の定義

上記の条件に当てはまらない限り、どのような処理内容であっても記述可能である。

constexpr関数に求められる条件は、C++11の[constexpr](chapter_1/core/constexpr.md)を参照。
以下にサンプルを示す。

```c++
constexpr int square(int x) {       // OK
    return x * x;
}

constexpr long long_max(int x) {    // OK
    return 2147483647;
}

constexpr int abs(int x) {          // OK
    if (x < 0) {
        x = -x;
    }

    return x;
}

constexpr int first(int n) {
    static int value = n;           // error variable has static storage duration.
    return value;
}

constexpr int uninit() {
    int value;                      // error variable is not initialized.
    return value;
}

constexpr int g(int x, int n) {     // OK
    int r=1;
    while (--n > 0) {
        r *= x;
    }
    return r;
}
```

##### constexprコンストラクタ
constexprコンストラクタの関数本体に記述可能な構文は、constexpr関数と同じである。
constexprコンストラクタの本体は、`= delete`されているか、constexpr関数本体の制約を満たして定義し、かつ、function-try-block を用いていないことが条件である。

constexprコンストラクタに求められる条件は、C++11の[constexpr](chapter_1/core/constexpr.md)を参照。

```c++
class X {
public:
    constexpr X(int x) : x_(x) { }
private:
    int x_;
};

class Y {
public:
    constexpr Y() {}
private:
    int y_ = 1;
};

class Z {
public:
    constexpr Z() try : z_(0) {      //error function-try-block cannot use in constexpr constructor.
    } catch (...) {
    }
}

int main() {
    constexpr X x(1);
    constexpr Y y;
}
```

