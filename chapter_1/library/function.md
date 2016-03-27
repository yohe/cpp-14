### *関数オブジェクト*
----
#### 概要
C++11では、いくつかの関数オブジェクトに関する機能が追加されている。また、それらにより代替可能なC++03時代の機能は非推奨機能となった。

まずは、新旧の移行について紹介する。

| C++03(Deprecated) | C++11 |
| -- | -- | -- |
| unary_function関連, binary_function関連 | function |
| mem_fun関連 | mem_fn |
| binder関連 | bind |

C++03で用途により分けられていた関数などは、一つに統合され、より使いやすくなっている。
以下に、上記で紹介した３つの機能について紹介する。
本節で解説する機能を使用するには、ヘッダファイル **function**をインクルードする。

##### bind
指定した関数に対して、特定の関数の引数を束縛(バインド)させた呼び出し可能なオブジェクトを返すテンプレート関数である。
呼び出し可能なオブジェクトとは、関数呼び出しと同じ構文を使用できるものを指す。

未バインドの引数が存在する場合は、オブジェクトの実行時(関数呼び出し時)に指定しなければならない。
未バインドの引数は、`std::placeholders`を使用することでオブジェクトの実行時に指定可能となる。
実行時に指定する第一引数の場合、`std::placeholders::_1`を使用し、第二引数の場合には`std::placeholders::_2`を使用する。
以降、数字を増やして対応する。使用可能なプレースホルダーの数は実装依存である。

bindに指定する関数には、通常の関数、メンバ関数へのポインタや呼び出し可能なオブジェクトを指定することができる。
メンバ関数を指定する場合には、インスタンスも含めて束縛するか、実行時に第一引数として指定しなければならない。
bindのテンプレート関数が返す型は未規定であるため、auto による型推論を使用するか、後述する**function**を使用する。

```c++
#include <functional>
#include <iostream>

void f(int a, int b) {
    std::cout << "f() " << a << ":" << b << std::endl;
}

class X {
public:
    int f(int a, int b) {
        std::cout << "X::f() "<< a << ":" << b << std::endl;
        x = 100;
        return a+b;
    }

    int x = 0;
};

static void f1(double a) {
    std::cout << "static void dfunc(double)" << std::endl;
}

class FuncObj {
public:
    void operator()(int a, int b) {
        std::cout << "FuncObj::operator()" << std::endl;
    }
};

int main() {

    using std::placeholders::_1;
    using std::placeholders::_2;

    auto func1 = std::bind(f1, _1);                         //OK f1の引数はプレースホルダとして実行時に指定
    func1(1.1);

    X x;
    auto func2 = std::bind(&X::f, std::ref(x), _1, _2);     //OK メンバ関数とインスタンスを束縛
    func2(1, 2);
    auto func3 = std::bind(&X::f, _1 , 1, 2);               //OK インスタンスは実行時に指定する
    func3(x);

    FuncObj obj;
    auto func4 = std::bind(obj, _1, _2);                    //OK 関数オブジェクトも指定可能
    func4(1, 2);

    return 0;
}
```

##### mem_fn
メンバ関数へのポインタを渡すことで、呼び出し可能なオブジェクト(callable object)を生成して返すテンプレート関数である。
mem_fnで生成したオブジェクトの型は未規定であるため、auto による型推論を使用するか、後述する**function**を使用する。
functionだけでも同じことが可能である。

```c++
#include <functional>
#include <iostream>

class X {
public:
   void f(int a) {}
};

int main() {
    auto func = std::mem_fn(&X::f);

    X x;
    func(x, 100);                       //OK インスタンスと関数の引数を渡す
    return 0;
}
```

##### function
任意の呼び出し可能なオブジェクトを受け入れるラッパークラスである。通常の関数や、メンバ関数なども格納することができる。
bind, mem_fnのテンプレート関数は、呼び出し可能なオブジェクトを生成するため、このfunctionに代入することもできる。
構築したfunctionオブジェクトは、通常の関数呼び出しと同じ構文を用いることで、構築時に渡した関数等を実行する。

```
void f(int a, int b) {
    std::cout << "f() " << a << ":" << b << std::endl;
}

class X {
public:
    int f(int a, int b) {
        std::cout << "X::f() "<< a << ":" << b << std::endl;
        return a+b;
    }
};

int main(int argc, char const* argv[])
{
    std::function<void(int, int)> func = f;
    f(10, 20);

    std::function<int(X&, int, int)> func2(&X::f);              //メンバ関数の場合は、第一引数にインスタンスを渡す
    X x;
    func2(x, 10, 20);
    return 0;

    
}
```
