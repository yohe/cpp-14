### **forward / move**
---
#### 概要
C++11にはr-value、ムーブセマンティクスという概念が追加されている事を記述した。
ムーブとはある変数の所有権を別の変数へ移譲する事であるため、一時的なテンポラリオブジェクトであるr-valueのみに適用可能である。
しかし実際には、使用しなくなるローカル変数(lvalue)を関数の引数へムーブしたい、関数内で作成したlvalueを関数の外へムーブしたい、
という用途が多い。 そこで、ムーブセマンティクスを示す *std::move* を利用する。

以下にstd::moveについて記載する。また、std::moveに似た機能を提供する *std::forward* についても記載する。

本節で解説する機能を使用するには、ヘッダファイル **utility**をインクルードする。

##### std::move
moveは、テンプレート関数であり引数に指定したパラメータをrvalue参照にキャストし、明示的なムーブセマンティクスを表現する。

以下に、上記の２パターンについて実装する場合の例を記載する。
* 関数呼び出し時のムーブ

```c++
#include <utility>>

class X{
public:
   X() {}
};

void func(X v) {
}

int main() {
    X x;
    func(std::move(x));
}
```

* 関数の戻り値をムーブ

```c++
#include <utility>>

class X{
public:
   X() {}
};

X func() {
    X x;
    return std::move(x);
}

int main() {
    X x = func();
}
```
関数の戻り値をムーブする場合の注意点として、戻り値の型は rvalue参照 ではないという点である。
関数を抜けた時点でローカル変数は破棄されるため、無効な参照となるためである。関数内のローカル変数をlvalue参照として返却できないのと同じ理屈となる。


##### std::forward
forwardは、テンプレート関数でムーブに近い機能を提供するが、
forwardを使用する主な用途は、テンプレート関数を定義する時である。

forwardが使われるのは、テンプレート関数の引数が*Universal reference*の時である。
Universal referenceは、与えたパラメータが、lvalue/rvalueかによってlvalue参照, rvalue参照のどちらにもなり得るためこのように呼ばれる。
ちなみに、Universal referenceは標準規格で定められた呼び名ではなく、通称である。
universal referenceについて知りたい場合は以下が参考になる。

[[Scott Meyers. "Universal References in C++11"](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)]<BR>
[[Scott Meyers. "Effective Modern c++"](https://www.oreilly.co.jp/books/9784873117362/)]<BR>

```c++
template <class T>
void func(T&& v) {}

int main() {
    int x;
    func(x);        // T=int& T&&=int&
    func(0);        // T=int  T&&=int&&
}
```

ここで、func()の内部で別の関数を呼び出す場合を考える。受け取った引数について、
lvalue参照として受け取った引数はlvalue参照で渡し、rvalue参照として受け取った引数はrvalue参照で渡したい。
このような要求で使用するのが、forwardである。forwardは、引数の参照タイプを保って別の関数へ渡すことができる。
この特性を*Perfect forwarding*と呼ぶ。
なお、rvalue参照で渡した変数は、forward後は不正となるため使用できなくなる。

```c++
template <class T, class A1. class A2>
shared_ptrT> factory(A1&& a1, A2&& a2) {
    return shared_ptrt<T>(new T(std::forward<A1>(a1), std::forward<A2>(a2)));
}

struct A {
    A(int& a, const double& b) {}
}

void g() {
    shared_ptrt<A> sp1 = factory<A>(2, 1.414);          //error rvalue参照はlvalue参照として渡せない
    int lvalue=2;
    shared_ptrt<A> sp1 = factory<A>(lvalue, 1.414);     //OK constなlvalue参照は rvalue を受け取ることが可能
}
```

