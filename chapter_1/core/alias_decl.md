### *エイリアス宣言*
---
#### 概要
以下のような構文をエイリアス宣言と呼ぶ。これはtypedefと同様に、指定した型名の別名を与える。エイリアス宣言はtypedefと同じ意味を持つ。
文法上ではエイリアス宣言の方が直感的でわかりやすいため、C++11では基本的にこちらを使用すれば良い。

`using 別名 = 型名;`

以下にusing宣言の使用例を示す。

```c++
using Point = std::pair<int, int>;      //OK
Point x;                                //OK

using func_p = void (*)(int);           //OK 関数ポインタの型
void func(int){}
func_p ptr=&func;                       //OK
```

エイリアス宣言のテンプレート版として、エイリアステンプレートがある。エイリアステンプレートはテンプレート引数を持ったエイリアス宣言である。
これは、テンプレートに対して完全または、部分特殊化を行った型に対して別名を定義する。

以下のように使用する。

```c++
template <class T>
using Point = std::pair<T, T>;

Point<int> p1;      //int型のPoint
Point<double> p2;   //int型のPoint
```

