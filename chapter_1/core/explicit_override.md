### *明示的な仮想関数オーバーライド*
---
#### 概要
C++03までのクラス定義では、意図的に仮想関数のオーバーライドを禁止する機能や、
オーバーライドしたいのに関数の宣言ミスなどによりオーバーライドされていない事を検知する機能がなかった。
また、クラス自体を意図的に継承させないといった機能も不足していた。
C++11では、クラス定義の際に使用できる新たなキーワードとして、"final", "override" が追加されている。
このキーワードはクラス定義時のみのものであり、そのほかの定義や宣言などでこれらの名前を使用しても ill-formed にはならい。

まずは、キーワード"final"について説明する。このキーワードをクラス定義のクラス名の後ろに指定することで、
そのクラスを継承不可に設定することができる。

```c++
class base { };
class final final : public Base { };        //OK クラスfinalに対し、継承不可に指定をする。
class ExtendFinale : public final { };      //error クラスfinalは継承不可
```

また、このキーワードを仮想関数宣言の後ろに指定することもできる。この場合、指定された仮想関数はオーバーライド不可に設定される。
```c++
class base {
public:
    virtual void func() = 0;
};

class derive : public base {
public:
    void func() final { }                   //OK 仮想関数funcをオーバーライド不可に指定する。
};

class override : public derive {
public:
    void func() { }                         //error 仮想関数funcは、クラスderiveにてオーバーライド不可となっている。
};
```

次に、キーワード"override"について説明する。このキーワードを仮想関数宣言の後ろに指定することで、仮想関数をオーバーライドすることを意味する。
このキーワードを指定したにも関わらずオーバーライドされない場合には ill-formed となる。
また、仮想関数に対する "final", "override" は同時に指定することも可能である。
```c++
class base {
public:
    virtual void func(int x) {}
};

class derive {
public:
    void fund(int x) override {}            //error 関数名が異なるためオーバーライドになっていない 
    void func() override {}                 //error 引数が異なるためオーバーライドになっていない
    void func(int x) override {}            //OK オーバーライドする
    void func(int x) final override {}      //OK オーバーライド、かつ継承先でオーバーライド不可に指定する.
}
```

