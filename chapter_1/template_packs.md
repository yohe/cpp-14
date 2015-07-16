### Variadic Template
---
#### **概要**
Variadic Templateとは、可変長引数テンプレートである。可変長テンプレートは、テンプレートパラメータの個数が可変であることを示す。
個数には、０個も含まれる。
Variadic Templateは、テンプレート宣言時に、以下のような形で"..."(エリプシス)を識別子の前に追加することで使用できる。

`template <class ... T>`

このように宣言されたとき、T をテンンプレートパラメータパックと呼び、T には引数として渡された型の全てが含まれる。また、テンプレートパラメータパックを関数の引数に指定した場合に、これを関数パラメータパックと呼ぶ。総称としてパラメータパックと呼ぶ

以下に宣言例を示す。

```c++
template <class ... Types>      //テンプレートパラメータパック
class Tuple {};

template <class ... Types>
void func(Types ... v) {}       //関数パラメータパック

Tuple<>          x0;
Tuple<double>    x1;
Tuple<char, int> x2;

func();
func(1, 2);
func(1, 2, 'a', "abc");
```

パラメータパックはそのままでは使用できず、パック展開を行うことで使用できるようになる。パック展開は、パターンと"..."を用いて記述する。パターンの形式は、パック展開を行う箇所に依存する。パック展開は基本的にコンマ区切りと同等の動作となる。

**関数パラメータの場合**
```c++
template <class X, class Y>
void impl(X x, Y y) {
    std::cout << x << "," << y << std::endl;
}
template <class Head, class ... Tail>       //先頭と残りに分離
void impl(Head h, Tail ... v) {
    std::cout << h << ",";
    impl(v...);                             //パック展開 ("v"がパターン)
}
template <class ... T>
void func(T ... v) {
    impl(v...);                             //パック展開 ("v"がパターン)
}

int main() {
    func(1, 2, 3) ;                         // 1, 2, 3
    return 0;
}
```
**テンプレートパラメータの場合**
```c++
template <class ... T>
class X : public T ... {                    //パック展開 ("T"がパターン)
};
```

**初期化子**
```
template <class ... T>
class X : public T ... {
public:
    X(): T() ... {}                         //パック展開 ("T()"がパターン) 
};

template <class ... T> void f(T ... rest);
template <class ... T> void g(T ... rest) {
    f(&rest ...);                           //パック展開 ("&rest"がパターン)

    std::vector<T> = {rest ...};            //パック展開 ("rest"がパターン)
}
```
