### *Hash container*
----
#### 概要
ハッシュ型の連想配列コンテナ(以下ハッシュコンテナ)である。要素への高速アクセス能力を持つ。最悪のケースではほとんどのオペレーションが線形時間となっている。
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


