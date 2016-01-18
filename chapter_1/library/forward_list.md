### *forward_list*
---
#### 概要
単方向リストを提供するコンテナである。C++03までは、listクラスによる双方向リストのみが提供されていた。
前方向走査のみが必要なケースでは、listクラスを使用すると余計なメモリを消費されることになり、手製のlinked-listが氾濫していた。
そこで、それらを一掃する目的でこのコンテナが追加された。

forward_listは、size()関数が提供されないことを除き、全ての一般コンテナ要件を満たす。
また、シーケンスコンテナ要件を満たさず、代わりとなるオペレーションが提供されている。
以下に、forward_listが提供するメンバ関数の一覧を示す。(オーバーロードの違いは無視する)

| メンバ関数名 | セマンティクス |
| -- | -- |
| assign | clear() + insert_affter() |
| get_allocator | |
| before_begin | begin() の前を示すイテレータを返す ( before_begin()++ == begin() ) |
| begin | 一般コンテナ要件と同じ |
| end | 一般コンテナ要件と同じ |
| cbegin | 一般コンテナ要件と同じ |
| cend | 一般コンテナ要件と同じ |
| empty | 一般コンテナ要件と同じ |
| max_size | 一般コンテナ要件と同じ |
| front | \*begin() |
| emplace_front | 要素を直接構築して先頭に挿入 |
| push_front | 指定した要素のコピーを先頭に挿入 |
| pop_front | erase_after(before_begin()) |
| insert_affter | 指定した位置の後ろに指定した要素のコピーを挿入 |
| emplace_affter | 指定した位置に後ろに直接構築した要素を挿入 |
| erase_affter | 指定した位置の要素を削除 |
| swap | 一般コンテナ要件と同じ |
| resize | コンテナのサイズを変更|
| clear | 全要素の削除|
| splice_after | 指定した位置の後ろにコンテナの要素を移植 |
| remove | 指定した値を持つ全要素を削除 |
| remove_if | 指定したPredicateを満たす要素を全て削除 |
| unique | 連続している重複した値を削除 |
| merge | 指定した順序、もしくは昇順で指定したリストを移植 |
| sort | リスト内をソート|
| operator== | 一般コンテナ要件と同じ |
| operator< | |
| operator!= | 一般コンテナ要件と同じ |
| operator> | |
| operator>= | |
| operator<= | |


このコンテナは、ヘッダファイル **forward_list**

```c++
#include <forward_list>

int main() {
    std::forward_list<int> flist {1,2,3};

    for(auto& i : flist) {
        std::cout << i << std::endl;
    }
}
```
