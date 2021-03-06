### *時刻(chrono)*
----
#### 概要
ライブラリchronoは、時間を扱うユーティリティ機能を提供する。
ライブラリには、時間の単位と時間間隔を表現する`duration`、エポック時間を起点にした時刻を示す`time_point`、時計機能を提供する `clock`で構成されている。
本節で解説する機能を使用するには、ヘッダファイル **chrono**をインクルードする。

##### duration
durationは単位時間を`ratio`を用いて表現し、ある時刻tから単位時間の何個分の時間間隔を示しているかを数値型を用いて管理する。
durationは、基本的な演算機能を持っている。

よく使う単位時間のdurationは、標準で定義されている。
以下に標準で定義されているdurationを示す。

| 単位 | 定義名 |
| -- | -- |
| ナノ秒 | std::chrono::nanoseconds |
| マイクロ秒 | std::chrono::microseconds |
| ミリ秒 | std::chrono::milliseconds |
| 秒 | std::chrono::seconds |
| 分 | std::chrono::minutes |
| 時 | std::chrono::hours |

また、durationは単位変換が可能になっている。精度が上がる場合にはそのまま代入するのみで良いが、
精度が下がる場合には、専用のキャスト`duration_cast`を用いる。

##### time_point
time_pointはエポック時間を起点としたある時刻を示す。また、time_pointは、duration型を持つことにより、時間の単位に合わせて柔軟に表現できるようになっている。
標準で規定されているclockを使用する場合には、clockごとにtypedefでtime_pointの型が指定されているためユーザが型を気にする必要はない。

エポック時間からの経過時間を取得するには、以下のメンバ関数を使用する。

`duration time_since_epoch() const`

##### clock
時計機能を提供するclockには、システム依存の `system_clock` 、安定した時計を提供する `steady_clock` 、高解像度の時計を提供する `high_resolution_clock` の３種類が標準で提供されている。

安定した時計とは、物理時間の進行のように常に時刻表現が増加する時計である。システム依存などの場合には、時刻変更により影響を受けてしまう場合がある。
clockには、安定した時計を提供するかを判定するメンバ変数があるため、これを用いてsteady_clock以外も安定しているかを確認することができる。

high_resolution_clockは、最も小さい単位の時計を提供する。
ただし、実装によっては system_clock or steady_clock の別名である可能性もある。

各clockは、以下のメンバ関数を使用することで、現在時刻を示すtime_pointを取得できる。

`static time_point now() noexcept`

各clockの内部型は実装依存となっており、以下のような定義名が存在する。

| 定義名 | 型 |
| -- | -- |
| rep | `unspecified` |
| period | `std::ratio<unspecified, unspecified>` |
| duration | `std::chrono::duration<rep, period>`
| time_point | system_clock : `std::chrono::time_point<system_clock>`<BR>other : `std::chrono::time_point<unspecified, duration>` |



以下にclockの使用方法を示す。

```c++
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;
    system_clock::time_point t1 = system_clock::now();

    // 時間の掛かる処理
    for(int i=0; i < 10000; i++) {
        std::cout << "* ";
    }
    system_clock::time_point t2 = system_clock::now();
    system_clock::duration elapsed = t2 - t1;

    std::cout << std::endl;
    if(std::ratio_equal<system_clock::period, std::nano>::value) {
        std::cout << "nanoseconds" << std::endl;
    }
    if(std::ratio_equal<system_clock::period, std::micro>::value) {
        std::cout << "microseconds" << std::endl;
    }

    std::cout << elapsed.count() << std::endl;

    // 単位変換 (duration_cast)
    milliseconds elapsed_milli = duration_cast<milliseconds>(elapsed);
    std::cout << elapsed_milli.count() << std::endl;

    // 単位変換 (精度が上がるためそのまま代入)
    microseconds elapsed_micro = elapsed_milli;
    std::cout << elapsed_micro.count() << std::endl;
    return 0;
```

以下に実行結果を示す。
```
nanoseconds
2357000
2
2000
```
