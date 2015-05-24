### *R-value reference*
---
#### 概要
R-value referenceとは、rvalueの参照を示す。rvalueとは一時的なオブジェクトや、参照でない関数の戻り値などの直ぐに破棄されるオブジェクトである。C++03で使用していた参照は、左辺値として使用できる変数(lvalue)に対するさる参照であり、C++11ではL-value referenceと呼ぶ。

R-value referenceの構文は次のように "&" を２つ重ねて定義する。
`int && rref = 5;`

```c++
int main(int argc, char const * argv) {
    int x;
    int && rref1 = x;       //error xはlvalue
    int && rref2 = 0;       //OK
}
```

このR-value referenceはどのような場面で使用するかというと、コピーのコストが非常に大きいオブジェクトの構築などがある。
以下のような状況を考える。

```c++

class HeavyCopy {
public:
   HeavyCopy() : data(new int[100000]) {}
   ~Heavycopy() { delete[] data; }
   HeavyCopy(const Heavycopy& v) {
       memcopy(data, v.data, 100000);
   }
};

Heavycopy func() {
    Heavycopy x;
    // x に関する処理
    return x;
}

int main(int argc, char const * argv) {
    Heavycopy tmp(func());
    return 0;
}

```

コンパイラによる最適化が何も無いと考えた場合、tmpはコピーコンストラクタが呼び出されるはずである。関数funcの戻り値は前述の通り、rvalueであり、直ぐに破棄されることになる。破棄されるのであれば、上記の例においてポインタdataの受け渡しのみでも良いはずである。そこで、このR-value referenceが役立つのである。rvalueのデータをlvalueへ移動させて構築が行われるため、これをムーブセマンティクスと呼ぶ。ムーブセマンティクスとは所有権の移譲である。ムーブセマンティクスを実現するために、クラスにはムーブコンストラクタ、ムーブ代入演算子が特別関数に追加された。

以下簡単にムーブコンストラクタ、ムーブ代入演算子について記載する。

* **ムーブコンストラクタ/ムーブ代入演算子**

```

#include <utility>

class X {
public:
    X() : data(new int[1000000]) {}

    X(X && rvalue) {                // ムーブコンストラクタ
        data = rvalue.data;
        rvalue.data = nullptr;
    }
    
    X& operator= (X && rvalue) {    // ムーブ代入演算子
        if ( this != &rvalue ) {
            data = rvalue.data;
            rvalue.data = nullptr;
        }
        return (*this);
    }

private:
    int* data;
};

X func();

int main() {

    X tmp(func());

    X a;

    a = std::move(tmp);
    X b(std::move(a));
}

```

ムーブコンストラクタについては単純にrvalue参照を受け取るコンストラクタである。ムーブ代入演算子は、コピー代入演算子と同じオペレータを用いて、引数がrvalue参照となる。それぞれ、rvalueを引数に渡す場合には特に問題無いが、lvalueを渡す場合は特別にムーブであることを明示するために、`std::move(X)`を用いる。ムーブとは所有権の移譲を意味するため、ムーブ以降のlvalueへのアクセスは制限される。制限内容はオブジェクトの型に依存する。`std::move`は、utilityヘッダをインクルードすることで使用可能となる。
