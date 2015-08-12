### *定数式(constexpr)*
---
#### 概要
キーワード"constexpr"を指定することで、対象をコンパイル時定数として扱えるようにする機能である。コンパイル時定数は、固定長配列のサイズ指定や、テンプレート引数に使用できる。また、constとの違いは、constはコンパイル時に定数であることが保証されていない点である。constexprが指定された対象は全てコンパイル時定数となる。C++11でのconstexpr関数は制約が厳しく使い勝手が悪いことから、C++14では制約が緩められている。
ここではC++11でのconstexpr条件とサンプルを示す。

以下にconstexprが指定可能な対象を示す。

 * 変数定義
 * 関数宣言,定義
 * テンプレート関数宣言,定義
 * リテラル型staticメンバ変数宣言

```c++
constexpr int square(int x);            //OK    関数宣言
constexpr int bufsz = 1024              //OK    変数定義

constexpr struct pixel {                //error 型宣言には指定できない
    int x;
    int y;
    constexpr pixel(int);               //OK    関数宣言(コンストラクタ)

constexpr pixel::pixel(int a)
    : x(square(a)), y(square(a))        //OK    変数定義
{}

constexpr int square(int x) {           //OK    定義
    return x*x;
}

int next(constexpr int x) {             //error パラメータには指定不可
    return x+1;
}

extern constexpr int memsz;             //error 変数定義ではない
```

変数にconstexprを指定する場合、以下の条件を満たす必要がある。
 * リテラル型
 * 初期化される

コンストラクタ以外の関数でconstexpr指定されたものをconstexpr関数と呼び、以下の条件を満たす必要が有る。
 * virtual関数でない
 * 戻り値がリテラル型
 * return文が一つで、パラメータがリテラル型

コンストラクタにconstexpr指定されたものをconstexprコンストラクタと呼び、以下の条件を満たす必要が有る。
 * パラメータはリテラル型
 * virtual基本クラスを持たない
 * tryブロックを持たない
 * 基本クラスはconstexprコンストラクタで初期化され、全ての非staticデータメンバ、基本クラスのサブオブジェクトは初期化されること

テンプレート関数(コンストラクタ含む)において、テンプレート関数がインスタンス化された際に、constexpr性を満足しない
場合、その関数はconstexprではなくなる。
