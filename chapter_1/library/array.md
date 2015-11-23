### *array*
---
#### 概要
固定長の配列を提供するクラスである。通常の配列と互換であり、加えて配列長の取得やアクセス系の関数が幾つか定義され、vectorと同じ感覚で使用できる。
{r|c}begin(), {r|c}end(), at(), size(), operator[], front(), back()などのアクセス関数が利用できる。
また、同じ値で埋めるfill(), 配列の中身を交換するswap()も定義されている。
通常の配列と互換であるため、メモリは連続して確保される。よって、`&a[n] == (&a[0] + n)`が保証されている。(0 <= n <= array.size())

C++11以降では、通常の配列と置き換えて使用することが想定されている。arrayは、ヘッダファイル *array* をインクルードすることで使用可能である。

以下に使用方法を示す。
```
#include <array>

class X {
public:
    X() : x(0) {}           // 初期値
    X(int a) : x(a) {}
    int x;
};

int main() {
    std::array<int, 5> arr {1, 2};          // 足りない要素は型の初期値が使用される

    int sum = 0;
    for(auto& v : arr) {
        std::cout << v << ",";
        sum += v;
    }
    std::cout << std::endl;

    for (int i=0; i < arr.size(); i++) {
        std::cout << arr[i] << ",";         // 添字アクセスも可能
    }

    std::array<X, 8> xarr {1, 2, 3, 4};     // ユーザ定義型も使用可能
}
```

