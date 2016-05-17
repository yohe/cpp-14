### *Contextually implicitly conversion*
---
#### 概要
C++14では、一部の文脈に依存した式での暗黙的な型変換に対する制約が緩められた。
一部の式では、クラス型のユーザ定義型変換オペレータが暗黙的に適用されるケースにおいて、候補となる変換オペレータは一つのみでなければならないという制約があった。

C++11までのC++では次のようなケースはill-formedとなる。
例えば、以下のようなクラスを考える。このクラスの型変換オペレータの定義は、一般によく使用されるC++のコーディングパターンである。

```c++
template< class T
          class = typename std::enable_if< std::is_arithmetic<T>::value ||
                                           std::is_pointer   <T>::value
                                         >::type
        >
class zero_init {
public:
    zero_init( )       : val ( static_cast<T>(0) ) {}
    zero_init( T val ) : val ( val ) {}

    // user-defined conversion operator
    operator T & ()       { return val; }
    operator T   () const { return val; }

private:
    T val;
};

void func() {
    zero_init<int*> p;
    p = new int(1);

    delete p;
}
```

この例のdelete式は、"p"の暗黙的型変換の候補が２種類存在する(Line 11, 12)ため、エラーとなる。
しかし、次のように記載した場合にはエラーとならない。

```c++
void func() {
    zero_init<int*> p;
    p = new int(1);

    delete (p+0);       //OK
    delete +p;          //OK
}
```

このように記述によって合法となるは、一部の式以外では式に対して一番適した型変換が採用されるからである。
本来必要の無い+演算子を適用したり、0を加えるだけで合法となるのはおかしいということで、制約が緩められた。

一つのみでなければならないという制約は、cv修飾子や参照に対するcv修飾子のバリエーションは許可された。但し、全て同一の型に変換されなければならない。

以下に制約が緩められた式について示す。

 * new 式
 * delete 式
 * リテラルクラス型を用いた整数定数式
 * switchの条件文

