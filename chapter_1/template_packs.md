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

##### 関数パラメータ
```c++
template <class ... Types>
void impl(Types ... args) {}                //OK パック展開 ("Types"がパターン)

template <class ... T>
void func(T ... v) {
    impl(v...);                             //OK パック展開 ("v"がパターン)
}

int main() {
    func(1, 2, 3) ;
    return 0;
}
```
##### テンプレートパラメータ
テンプレートパラメータの引数宣言時にパック展開可能である。引数宣言時にパック展開された場合、展開後の引数は非型テンプレート引数として扱われる。
```c++
template <class ... packs>
class Outer{
public:
    template <packs ... args>           //OK packs がパターン(非型テンプレート引数)
    class Inner {};
};

int main() {
    Outer<char, int, long>::Inner<'a', 1, 5L> a;        //OK Outerで指定した型に対する値をパラメータパックとして指定
}
```

テンプレートパラメータの引数指定時にパック展開可能である。
```c++
template <class ... T>
class Tuple {};

template <class ... T>
class X {
    Tuple<T...> t;                      //OK パック展開 ("T"がパターン)
    Tuple<T*...> t_p;                   //OK パック展開 ("T*"がパターン)
}

int main() {
    
}
```

##### 初期化子
初期化子を記述する箇所でもパック展開を行うことが可能である。
```c++
template <class ... T>
class X : public T ... {
public:
    X(): T() ... {}                         //OK パック展開 ("T()"がパターン) 
};

template <class T, T ... values>
class Vec {
public:
    std::vector<T> a { values ... };        //OK パック展開 ("values"がパターン)
};

template <class ... T> void f(T ... rest);
template <class ... T> void g(T ... rest) {
    f(&rest ...);                           //OK パック展開 ("&rest"がパターン)

    std::vector<T> = {rest ...};            //OK パック展開 ("rest"がパターン)
}
```

##### ベースクラス指定
ベースクラスを指定する箇所でもパック展開を行うことが可能である。

```c++
template <class ... T>
class X : public T ... {                    //OK パック展開 ("T()"がパターン) 
};
```

##### 例外指定
関数、メンバ関数宣言時の例外指定を行う箇所でもパック展開が可能である。
ただし、throw () による例外指定は非推奨機であるため、詳細は記載しない。

##### キャプチャリスト
ラムダ関数を定義する際のキャプチャ指定時にもパック展開が可能である。

```c++
template <class ... Types>
void g(Types ... args) {

template <class ... Types>
void f(Types ... args) {
    auto a1 = [args...] () { g(args...); };     //OK [ ]内の "args"がパターン(コピーキャプチャ)
    auto a2 = [&args...] () { g(args...); };     //OK [ ]内の "&args"がパターン(参照キャプチャ)
}
```

##### その他
上記以外の記述箇所(Attribute, アライメント指定)にも指定可能だが、ほとんど使用するケースがないので
省略する。


さて、関数などの引数で指定されたパラメータパックの各パラメータをどのように扱うかを説明する。
まず、一つの方法として固定数の引数が定義された関数にリダイレクトすることである。これは簡単に、以下のようになる。
```c++
template <class X>
void func_impl(X x) {}

template <class X, class Y>
void func_impl(X t, Y y) {}

template <class X, class Y, class Z>
void func_impl(X t, Y y, Z z) {}

template <class ... T>
void func(T ... args) {
    func_impl(args);
}

int main() {
    func(1);            //OK
    func(1, 2);         //OK
    func(1, 2, 3);      //OK
}
```

もう一つは、再帰を用いる方法である。これは上記の方法に比べ複雑であるが、汎用性が高い。
```c++
template <class X>
X sum(X a, X, b) {                  //再帰停止条件
    return a + b;
}

template <class X, class ... Tail>
X sum(X a, Tail ... tail) {         //先頭と、残りに分割
    return a + sum(tail...);        //残りについて再帰を適用
}

int main() {
    sum(1, 2, 3, 4, 5);
}
```
同様の方法で、テンプレートクラスの場合でもパラメータパックから個別に型を取り出して扱うことも可能である。

```c++
template <class ... args>
class Base;

template <class T, class ... Tail>
class Base<T, Tail...> {
public:
    T head;
    Base<Tail> Tail;
};

template <>
class Base<> {};

int main() {
    Base<int, long> b;
    std::cout << typeid(b.head).name() << std::endl;                //OK int
    std::cout << typeid(b.tail.head).name() << std::endl;           //OK long
    std::cout << typeid(b.tail.tail.head).name() << std::endl;      //error Base<>はheadを持たない
}
```


最後に、パラメータパックに含まれるパラメータ数を取得する方法を紹介する。パラメータパックのパラメータ数を取得するために、新たなオペレータ"sizeof..."が追加されている。
```c++
template <class ... Types>
class X {
    const size_t value = sizeof...(Types);
};
```
