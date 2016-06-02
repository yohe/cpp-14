### *make_unique*
---
#### 概要
C++11でスマートポインタが追加されている。追加されたスマートポインタのうち、`shared_ptr`に対してのみ構築用のヘルパ関数が提供されていた。
C++14で、unique_ptrに対しても構築用ヘルパ関数`make_unique`が追加されたため、これにより殆どのケースでnew/deleteをソースコード上から排除できるようになる。

make_unique関数は配列にも対応している。

以下にサンプルを示す。
```c++
#include <memory>

int main() {
    std::unique_ptr<int>   p1 = std::make_unique<int>();
    std::unique_ptr<int[]> p2 = std::make_unique<int[]>(5);

    for(int i = 0; i < 5; ++i) {
        std::cout << p2[i] << " ";
    }
    std::cout << std::endl;             // 0 0 0 0 0
}
```

