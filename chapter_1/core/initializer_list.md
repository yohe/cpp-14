### *統一初期化構文*
---
#### 概要
統一初期化構文は初期化式であり、統一初期化構文の構文は全ての変数に対する初期化式、もしくはコンストラクタ呼び出しとして使用可能である。故にこれを統一初期化構文と呼ぶ。
統一初期化構文は、以下のように波カッコの構文で記述する。ここでの初期化リストとは、値のコンマ区切りリストである。

```c++
型 名前 {初期化リスト}
型 名前 = {初期化リスト}
```

C++03では波カッコによる初期化構文は配列型、集約型に対してのみ使用可能であった。C++11ではこれが拡張されユーザ定義型にも適用可能になり、統一的な初期化構文として使用できるようになったのである。
また、変数宣言の際にデフォルトコンストラクタを丸カッコで記述すると関数宣言と誤認識する問題があった。この問題に対し、統一初期化構文を用いることで関数宣言と誤認識する問題を回避することが可能になる。
ただしこの構文を用いる場合、型のサイズが小さくなるような変換を暗黙的に含む場合には ill-formed として扱われる



```c++
struct X{};

X f();          //warning デフォルトコンストラクタの呼び出しを意図するが、関数宣言として解釈。
X f{};          //OK デフォルトコンストラクタの呼び出し

class hoge {
public:
    hoge(int a, double b) : a{a}, b{b} {
    }

private:
    int a;
    double b;
};

hoge h1 = {1, 1.3};     //OK コンストラクタ呼び出し
hoge h2{1, 1.3};        //OK コンストラクタ呼び出し
hoge h3{1.0, 1.3};      //error double -> int への変換は不可
hoge h4(1.0, 1.3);      //OK

struct Y {              //集約型
    int a;
    int b;
};

Y y0 {1, 2};                            //OK 型Yを初期化リストで初期化
Y y1 = {1, 2};                          //OK 型Yを初期化リストで初期化
Y y2[3] = { {1,2}, {3, 4}, {5, 6} };    //OK 型Yの配列を初期化リストで初期化
```

この初期化構文は上記の通り、配列にも適用可能である。配列に対して適用する場合、以下のようなルールで扱う。
 * 配列の要素数が指定されている
  * 初期化リストの要素数が配列の要素数より少ない場合は、指定された順で先頭から初期化され、足りない要素数は zero-initialize される
  * 初期化リストの要素数が配列の要素数より多い場合は、ill-formed
  * 初期化リストの要素数と配列の要素数が同じ場合は、指定された順で先頭から初期化
 * 配列の要素数が指定されていない
  * 初期化リストの要素数の配列として宣言されたものとして振舞う。この際、指定された順で初期化。

```c++
int a[3] = {1, 2, 3};       //OK
int b[3] {1, 2, 3};         //OK

int c[3] = {1, 2};          //OK c[0]=1, c[1]=2, c[2]=0
int d[3] = {1, 2, 3, 4};    //error

int e[] = {1, 2, 3};        //OK 配列のサイズは3
```

また、面白いことにクラス型の構築において、代入構文のような記述がコンストラクタ呼び出しとなる点である。
但し、初期化リストの前に型名を記述する場合には、コピーコンストラクタも動作する。
※ 型名を記述した場合であっても、基本的にコンパイラの最適化によってコンストラクタのみとなる。

```c++
class X {
public:
    X(int a, double b) : a{a}, b{b} {
        std::cout << "constructor" << std::endl;
    }
    X(const X& rhs) {
        std::cout << "copy constructor" << std::endl;
    }
private:
    int a;
    double b;
}

int main() {
    X x = { 1, 1.0 };
    X x = X{ 1, 1.0 };
    return 0;
}
```
constructor
constructor
copy constructor
```
```


初期化リストの特殊系として、`std::initializer_list<T>`を使用する方式がある。
これについては、ライブラリの項目を参照。
