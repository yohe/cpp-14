### *exchange*
---
#### 概要
本機能は、単純に変数の値を、新しい値と交換するだけの機能である。
`std::move`を用いて実装されているので、ムーブに対応した型であれば効率的な交換が行われる。

本機能は、ヘッダファイル`utility`に追加されている。

以下に実行例を示す。
```c++
#include <utility>

int main() {
    int i, j;
    i = 1;
    j = 2;
    std::exchange(i, j);        //OK i is 2.
    std::exchange(i, 3);        //OK i is 3.
    return 0;
}
```

