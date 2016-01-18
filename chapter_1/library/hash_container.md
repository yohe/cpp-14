### *ハッシュコンテナ*
----
#### 概要
ハッシュ型の連想配列コンテナ(以下ハッシュコンテナ)である。要素への高速アクセス能力を持つ。最悪のケースではほとんどのオペレーションが線形時間となる。
ハッシュコンテナは、ハッシュ化可能なキーを扱う。マップの場合にはキーに加えて、キーに紐づく値も扱う。
なお、ハッシュコンテナは、順序性を持たないため順序性が必要となる場合は、ハッシュ型ではない連想コンテナを利用する。
ハッシュ型の連想配列コンテナとして、以下のクラスが追加されている。

| コンテナ | 名前 |
| -- | -- |
| ハッシュ型セット | std::unordered_set |
| ハッシュ型マップ | std::unordered_map |
| ハッシュ型マルチセット | std::unordered_multi_set |
| ハッシュ型マルチマップ | std::unordered_multi_map |

上記のハッシュコンテナを使用するには、ヘッダファイル *unorderd_set*, *unorderd_map* をそれぞれインクルードする事で使用可能となる。

ハッシュコンテナが追加される事により、ハッシュクラステンプレートが追加されており、プリミティブな型と幾つかのライブラリのクラスがハッシュ化に対応している。
ハッシュ化に対応している型をキーに使用する事ができる。そのため、自作のクラスをキーに使用したい場合は、ハッシュ化に対応させる必要が有る。
ハッシュ化対応は、キーに指定した型を引数に受け取り、size_t型を返却する関数オプジェクトを用いる方法、もしくは`std::hash<T>`を特殊化する２つの方法がある。

以下にハッシュ化の定義例を示す。
* 関数オブジェクト

```c++
#include <unordered_map>
struct HashKey {
    int value1;
    int vlaue2;
};
struct Hasher {
    size_t operator()(const HashKey& k) const {
        std::string str = std::to_string(k.value1) + "_" + std::to_string(k.value2)
        std::hash<std::string> h;
        return h(str);
    }
};

int main () {
    std::unordered_map<HashKey, int, Hasher> m;
    return 0;
}
```

* std::hash<T>の特殊化

```c++
#include <unordered_map>
struct HashKey {
    int value1;
    int vlaue2;
};
namespace std {
    template <>
    struct hash<HashKey> const {
        size_t operator()(const HashKey& k) {
            std::string str = std::to_string(k.value1) + "_" + std::to_string(k.value2)
            std::hash<std::string> h;
            return h(str);
        }
    };
}

int main () {
    std::unordered_map<HashKey, int> m;
    return 0;
}
```

ハッシュ型の連想配列コンテナは、通常の連想配列コンテナと同じように使用する事が可能である。いかに使用方法を示す。

```c++
#include <unordered_map>
#include <iostream>

int main() {
    std::unordered_map<char, int> u_map {
        {'A', 1},
        {'B', 2},
        {'C', 3},
        {'D', 4},
        {'E', 5},
        {'F', 6},
        {'Z', 7},
    };

    for (const auto& k : u_map) {
        std::cout << "key=" << k.first << ", value=" << k.second << std::endl;
    }

    u_map.insert(std::make_pair('G', 7));
    u_map['Z'] = 8;

    std::cout << "-----------------" << std::endl;

    for (const auto& k : u_map) {
        std::cout << "key=" << k.first << ", value=" << k.second << std::endl;
    }

    return 0;
}
```

いかに実行結果を示す。
```
key=F, value=6
key=E, value=5
key=Z, value=7
key=D, value=4
key=C, value=3
key=B, value=2
key=A, value=1
-----------------
key=G, value=7
key=F, value=6
key=E, value=5
key=Z, value=8
key=D, value=4
key=C, value=3
key=B, value=2
key=A, value=1
```

また、ハッシュ型連想配列コンテナには、通常の連想配列コンテナにはない、*bucket* という概念がある。
ハッシュ型連想配列コンテナの要素は、このbucketに格納され、同じハッシュ値を持つ要素は、同じbucketに格納する事が保証されている。
ただし、要素の追加などを行うことで bucket が追加される場合がある。ハッシュコンテナは、各bucketに対して均等に要素が登録されるように調整する。
よって、bucket数が変化した場合、ハッシュコンテナ内の要素が再配置される。
再配置(rehash)が行われると、要素間の順序が変更され、要素が格納されるbucketも変更される。つまり、再配置が行われると、その時点のイテレータは無効になる。
