### *decltype(auto)*
---
#### 概要
C++11で、decltype(式)による型指定の構文が追加された。
decltypeを用いることでテンプレートを使用した場合やautoで宣言された変数の型を意識せずに記述する事が可能になった。 
但しC++11の時点では、以下のように変数宣言時に型判定で使用する式を、左辺(decltypeの括弧内の式)と右辺の両方に記述する必要があった。

```c++
decltype(a + b) integer = a + b;
```

しかし、decltype(auto)を変数宣言時に用いると、右辺のみに記述するだけで良くなる。
この時の変数の型は、初期化式をdecltype()に渡した時に判定される型と同じとなる。
判定される型のルールは、[auto](chapter_1/core/auto.md)のdecltype関連を参照。

```c++
template <class T, class U>
void func(T t, U u) {
    decltype(t + u) v1 = t + u;     // v1 is double

    // decltype(t + u) v2 = t + u と等価
    decltype(auto)  v2 = t + u;     // v2 is double

    int i;
    decltype(auto)  v3 = i;         // v3 is int
    decltype(auto)  v4 = (i);       // v4 is int&
}

int main() {
    func(char(), double());
    return 0;
}
```

