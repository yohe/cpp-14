### *コンストラクタ移譲*
---
#### 概要
C++11では、コンストラクタのメンバ初期化子にて、そのクラスが定義する他のコンストラクタを記述することができ、コンストラクタの処理を他のコンストラクタへ移譲することができる。移譲元のコンストラクタを移譲コンストラクタ(delegating constructor)と呼び、移譲先のコンストラクタをターゲットコンストラクタ(target constructor)と呼ぶ。
移譲元のコンストラクタは、メンバ初期化子にターゲットコンストラクタのみが指定でき、ターゲットコンストラクタの処理が完了後に、移譲元コンストラクタの関数部が実行される。

直接的、間接的問わずに移譲元のコンストラクタがターゲットコンストラクタとして呼び出される場合、つまりコンストラクタの移譲が循環する場合は ill-formed となる。

```c++
#include <iostream>

class X {
public:
    X() : X(0) {
        std::cout << "X()" << std::endl;
    }
    X(int) : X('a') {
        std::cout << "X(int)" << std::endl;
    }
    X(char) {
        std::cout << "X(char)" << std::endl;
    }
};

int main() {
    X x;
    return 0;
}
```

上記のプログラムを実行した場合の処理結果
```
X(char)
X(int)
X()
```

