### *コンパイル時整数列*
---
#### 概要
C++11で可変長テンプレート機能が追加されている。この機能を用いることで、
以下のようにパラメータの詳細を機にすることなく、他の関数へ転送することができるようになった。
この時、`t`はパラメータパックであり、`t...`とすることでパック展開を行うことができる。
```c++
template <class... T>
void f(T... t) {        //tは関数パラメータパック
    expr(t...);         //パック展開
}
```

ここで、以下に示すようなパラメータパックが他の型の一部であるような場合、引数はパラメータパックとして扱われない。
ゆえに、前述の例のように直接的にパック展開を行うことができない。


```c++
template <class... T>
void f(std::tuple<T...> t);
```

この例のようにタプルを展開させるためには、関数を再帰的に呼びさして一つずつ展開させる必要があった。
これを簡単化させるために、コンパイル時整数列の機能が追加された。

```c++
template <int...> struct index_sequence;
```

上記のようなテンプレートクラスで表現され、テンプレート引数が<0, 1, 2, 3>となるような型が生成される。
このクラスの生成用にヘルパクラスが定義されており、`make_index_sequence`テンプレートクラスを用いる。
これのテンプレート引数に生成したい個数の数を指定することで、0, 1, ..., Nまでの整数列が生成される。

これらを用いることで、タプルの展開を以下のように簡略化できる。

```c++
template <typename Tuple, size_t... I>
voud f(Tuple t, index_sequence<I...>)
{
    expr(std::get<I>(t)...);
    //以下のように展開される
    //expr(std::get<0>(t), std::get<1)(t), ...
}

template <typename Tuple,
          typename Indices = make_index_sequence<std::tuple_size<Tuple>::value>>
void tuple_expand(Tuple args) {
    f(tuple, Indices());
}
```

また、arrayからtupleへの変換も直感的に実現できる。

```c++
template <typename Array, size_t ...I>
auto a2t_(const Array& a, index_sequence<I...>)
 -> decltype(std::make_tuple(a[I]...))
{
    return std::make_tuple(a[I]...);    //std::make_tuple(a[0], a[1], ...)
}

template <typename Array, size_t N, typename Indices = make_index_sequence<N>>
auto a2t(const Array& a, index_sequence<I...>)
 -> decltype(a2t_(a, Indices()))
{
    return a2t_(a, Indices());
}

```

