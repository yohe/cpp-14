### *auto*
---
**概要**

C++03で自動変数(auto変数)を宣言する際の記憶クラス指定子であったが、指定されることはないためC++11において廃止となり、型推論を示す型として再定義された。
autoを変数を宣言する際の型として指定すると、変数の型は、型推論により宣言に続く初期化式の結果の型として宣言されたものとして振舞う。
従って、autoを指定した変数宣言には、必ず初期化式が必須となる。

また、このキーワード"auto"は、関数の戻り値の型推論にも使用することができる。但しreturn文からの型推論ではなく、
trailling return typeと呼ばれる後置形の型指定の式の結果から推論する。

**サンプル**
#### 変数宣言

```
class X {
public:
    X() {}
    X(const X& lhs) {
    }
};

X f();
X& f_ref();

auto func(int i, int j) -> int {        // trailing return type
    return i+j;
}

int main(int argc, char** argv) {
    auto i = 1;             // int
    auto f = 1.0f;          // float
    auto d = 10.0;          // double

    auto v = f();           // X
    auto & v_ref = f_ref(); // X &
}
```
