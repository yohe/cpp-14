### *通常関数の戻り値型推論*
---
#### 概要
C++11において、関数の戻り値の型推論はラムダ関数にのみ制限されていた。加えて、正しく型推論させるためには、return文一つのみでラムダ関数が記述されている必要があった。
C++14では、このラムダ関数のみという制約と、関数本体がreturn文一つのみでなければならないという制限が取り除かれた。

これにより、通常関数の戻り値の型宣言に`auto`, もしくは`decltype(auto)`を使用する事が可能になった。

型推論において、型推論に用いられる式に推論対象の実体が使用されている場合、そのプログラムはill-formedである。
しかし、戻り値の型を推論させる関数の中で、その関数を呼び出すようなreturn文があったとしても、残りの関数本体の中で戻り値の型が確定できる場合には、許可される。

以下の通りである。

```c++
auto v1 = v1;           //NG 型推論の式に推論対象が含まれる
auto v2 = (int)v2;      //NG

auto fibonacci(int num) {
    if (num == 0) return 0;                             //OK return type is int.
    if (num == 1) return 1;                             //OK return type is int.

    return fibonacci(num - 2) + fibonacci(num - 1);     //OK
}

auto fibonacci_(int num) {
    if (num >= 2) 
        return fibonacci(num - 2) + fibonacci(num - 1); //OK

    if (num == 0) return 0;                             //OK return type is int.
    if (num == 1) return 1;                             //OK return type is int.

}

decltype(auto) fibonacci__(int num) {       //OK
    if (num >= 2) 
        return fibonacci(num - 2) + fibonacci(num - 1);

    if (num == 0) return 0;
    if (num == 1) return 1;

}
```

前述の通り、ラムダ関数についても複数のステートメントを記述可能となり、複数return文可能となっている。

```c++
int main() {
    auto func = [&func](int n) {
        if (n == 0) return 0;                   //OK return文のみでなくても良い。
        if (n == 1) return 1;                   //OK 複数のreturn文を記述できる。
        // return func(n - 2) + func(n - 1)     //error 推論の初期化式に自身が含まれる
    }
    return 0;
}
```
ラムダ関数は、戻り値の型推論を使用して再帰呼び出しすることが出来ない。再帰したい場合は、funcの型を明示的に指定(`std::function<int(int)>`)する必要がある。

戻り値の型推論はテンプレート関数にも適用することが可能である。
戻り値の型推論を用いて宣言された関数、及びテンプレート関数は、型推論を用いずに宣言された関数とは別扱いとなる。 
つまり、戻り値の型推論を用いた関数、及びテンプレート関数の再宣言や特殊化を行う際は、最初の宣言時に指定したプレースホルダー(auto or decltype(auto))を使用しなければならない。
また、プレースホルダーを使用して関数を宣言する場合、その関数は`virtual`にすることは出来ない。

```c++
auto f();
auto f() { return 42; }     //return type is int
auto f();                   //OK

int f();                    //error f()と曖昧になる

template<typename T> auto g(T t) { return t; }  //#1
template auto g(int);                           //OK return type is int
template char g(char);                          //error マッチする関数がない。autoを指定していない
template<> auto g(double);                      //OK 関数の特殊化の前方宣言。関数本体がないため戻り値の型はまだ未定
template<> auto g(double d) { return 0; }       //OK return type is int
```

