### *scoped enum*
---
#### 概要
C++03までのenumは、宣言したスコープに対して列挙子を持ち込む、スコープ無しの機能であった。これは、C++11では*unscoped enumerator*と呼ぶ。
この従来のenumに加え、C++11では *scoped enumerator*が追加された、このscoped enumeratorを使用することで、列挙子はenum名のスコープに閉じ込められる。
これにより、識別子名の衝突が緩和される。

scoped enumeratorを使用する場合には、以下のように記述する。２種類の記法が存在するが、シンタックス上の違いのみである。

`enum class enum名 { 列挙子リスト };` or `enum struct enum名 { 列挙子リスト };`

この記法を用いる場合、enum名は必須である(unscoped enumeratorはオプショナル)。また、列挙子の暗黙的な整数変換が行われなくなり、
より厳格な型チェックが行われるようになる。そのため、C++11では基本的にscoped enumratorのみを利用していけば良い。

```c++
enum global {
    abc         //abc はグローバルスコープで定義される
};


class x {
public:
    enum xscope {
        abc     //abc はクラスxのスコープで定義される
    };
};

enum class enumscope {
    abc         //abc はenumscopeのスコープで定義される
};

int main(int argc, char const* argv[]) {
    ::abc;                          //enum global
    x::abc;                         //enum xscope
    enumscope::abc;                 //enum class enumscope


    int value1 = ::abc;             //OK 列挙子->int
    long value2 = x::abc;           //OK 列挙子->long
    int value3 = enumscope::abc;    //error 整数変換は不可
    return 0;
}
```

scoped enumeratorに加え、C++11では全てのenum宣言に対して明示的に列挙子の型を指定することができるようになった。scoped enumeratorの場合、省略時はint型となる。
unscoped enumeratorの場合には不定である。

`enum [class|structor] enum名 : 型指定子 { 列挙子リスト };`

