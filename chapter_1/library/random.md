### *乱数(random)*
---
#### 概要
C++03における乱数生成は、Cのrand関数を利用したstd::rand()を提供していた。
しかし、このrand関数は質が悪い、スレッドセーフではないなどの問題があったため、C++11で新たに乱数を生成するライブラリが新たに追加された。

乱数には、擬似乱数と非決定的乱数というものが存在する。
擬似乱数とは、乱数生成器の初期化時に指定する初期値によって、生成される乱数を予測可能なものを指す。一般に初期値をシードと呼ぶ。
逆に、非決定的乱数とは完全に予測不可能な乱数である。 この乱数は、放射性同位元素の崩壊時間、ツェナダイオードのノイズや熱雑音、温度などを計測することで得ることができる。
非決定的乱数は擬似乱数と比べ乱数生成に時間が掛るため、基本的には擬似乱数を利用する。シードを設定してあげることで実用上の問題はない。

乱数を使ったシミュレーションを行う場合、値の範囲や値の発生率を表す確率分布を考える必要が有る。例えば、サイコロの出る目を扱う場合は1から6の範囲で一様分布。テストの平均点が60点である場合の生徒が取るであろう点数を扱う場合は0から100の範囲で正規分布を用いる。

C++11では上記で説明した乱数に関する機能が提供されている。
擬似乱数(random number engine)、非決定的乱数(random_device)、確率分布(distribution)について以下で説明する。

本節で解説する機能を使用するには、ヘッダファイル **random**をインクルードする。

##### 擬似乱数
擬似乱数とは、数学的なアルゴリズムにより生成される乱数であり、乱数生成時のパラメータから次に生成される乱数が決定的であるような乱数である。
シード値を設定する事で、生成される乱数列を変更する事が可能である。
以下に擬似乱数生成器の一覧、および簡単化のために用意されているtypedefを紹介する。

| アルゴリズム | テンプレートクラス名|
| -- | -- |
| 線形合同法 | linear_congruential_engine |
| メルセンヌ・ツイスタ法 | mersenne_twister_engine |
| キャリー付き減算法 | subtract_with_carry_engine |

| typedef | 概要 |
| -- | -- |
| minstd_rand0 | 線形合同法における A=16807, B=0, M=2^31-1 |
| minstd_rand | 線形合同法における A=48271, B=0, M=2^31-1 |
| mt19937 | メルセンヌ・ツイスタにおける周期が2^19937-1となる生成法 |
| mt19937_64 | 上記の64bit版 |
| ranlux24_base | キャリー付き減算法における S=10, R=24, M=2^24 |
| ranlux24 | discard_block_engine<ranlux24_base, 223, 23> |
| ranlux48_base | キャリー付き減算法における S=5, R=12, M=2^48 |
| ranlux48 | discard_block_engine<ranlux48_base, 389, 11> |
| knuth_b | shuffle_order_engine<minstd_rand0, 256> |
※ "std::"は省略


擬似乱数の生成アルゴリズムは複数あるため、ここではメルセンヌ・ツイスタ法の例を紹介する。メルセンヌ・ツイスタ法はパラメータが多く使用が難しい。
そのため、周期2^19937-1であるメルセンヌ・ツイスタ(MT19937)が定義されており、例ではこれを使用する。
なお、メルセンヌ・ツイスタは生成される乱数の周期がとてつもなく長いが、624回の生成値を取得する事で以降の生成値を予測可能になることが知られている。そのため、ハッシュ化関数を通さない限り暗号化に使用してはならない。

```c++
#include <random>
#include <iostream>

int main(int argc, char const* argv[])
{
    std::mt19937 mt;                    // デフォルトのシード値を使用
    std::cout << mt() << std::endl;
    std::cout << mt() << std::endl;     // 次の呼び出し時は異なる値が取得できる

    mt.seed(1);                         // シード値の再設定
    std::cout << mt() << std::endl;

    std::mt19937 mt_1(100);             // 指定したシード値で構築
    std::cout << mt_1() << std::endl;
    return 0;
}
```


##### 非決定的乱数
非決定乱数とは、ハードウェア情報を乱数源とした予測不可能な値を生成する乱数である。
C++11では、非決定的乱数に相当するrandom_deviceクラスがある。このクラスを使用することで完全にランダムな乱数を取得できるようになる。
ただし、前述のように特殊な装置を必要とする。そのため、このような乱数を提供できない環境では、擬似乱数を内部で使用する可能性もある。

以下に、random_deviceの使用例を示す。

```c++
#include <random>
#include <iostream>

int main() {
    std::random_device rd;
    std::cout << rd() << std::endl;
}
```

##### 確率分布
確率分布とは、ある特定の値の範囲において、各値がどれくらいの確率で発生するかを与えるものである。
C++11には、様々な種類の確率分布が定義されている。例えば離散型で、一様分布を生成する uniform_int_distribution, TrueかFalseの2値のみを扱うbernoulli_distribution。
連続型では、一様分布を生成する uniform_real_distribution, 正規分布を生成する normal_distrobutionなどがある。
以下に、確率分布一覧を示す。

| 種別 | 名前 | 分布型 | テンプレートクラス名 |
| :---: | --- | :---: | --- |
| 一様分布 | - | - |
| - | 離散一様分布 | 離散型 | uniform_int_distribution |
| - | 連続一様分布 | 連続型 | uniform_double_distribution |
| ベルヌーイ分布 | - | - | - |
| - | ベルヌーイ分布 | 離散型 | bernoulli_distribution |
| - | 二項分布 | 離散分布 | binomial_distribution |
| - | 負の二項分布 | 離散分布 | negative_binomial |
| - | 幾何分布 | 離散分布 | geometric_distribution |
| ポアソン分布 | - | - | - |
| - | ポアソン分布 | 離散型 | poisson_distribution |
| - | 指数分布 | 連続型 | exponential_distribution |
| - | ガンマ分布 | 連続型 | gamma_distribution |
| - | ワイブル分布 | 連続型 | weibull_distribution |
| - | 極値分布 | 連続型 | extreme_value_distribution |
| 正規分布 | - | - | - |
| - | 正規分布 | 連続型 | normal_distrobution |
| - | 対数正規分布 | 連続型 | lognormal_distribution |
| - | カイ二乗分布 | 連続型 | chi_squarea_distribution |
| - | コーシー分布 | 連続型 | cauchy_distribution |
| - | F分布 | 連続型 | fisher_f_distribution |
| - | T分布 | 連続型 | student_t_distribution |
| サンプリング分布 | - | - | - |
| - | 離散分布 | 離散型 | discreate_distribution |
| - | 区分定数分布 | 連続型 | piecewise_constant_distribution |
| - | 区分線形分布 | 連続型 | piecewise_linear_distribution|

基本的に離散型は整数型(デフォルトはint)を用いて、連続型は浮動小数点型(デフォルトはdouble)を戻り値として使用する。また、確率分布の適用方法は、使用したい分布に対して使用する乱数生成器を渡すだけである。

以下に、離散型の一様分布を使用する例(サイコロを1000回振った場合の例)を示す。

```c++
#include <random>
#include <iostream>
#include <array>

int main() {
    std::uniform_int_distribution uid(1, 6);
    std::random_device rd;
    std::mt19937 mt(rd())
    
    // サイコロを1000回振った時に出た目のシミュレーション
    std::array<int, 6> ret {};
    for(int i=0; i<1000; i++) {
        ret[uid(mt)-1]++;
    }

    for(int i=0; i<6; i++) {
        std::cout << "[" << i+1 << "] : " << ret[i] << std::endl;
    }

    return 0;
}
```

以下に出力結果を示す。

```
[1] : 166
[2] : 161
[3] : 174
[4] : 183
[5] : 154
[6] : 162
```
