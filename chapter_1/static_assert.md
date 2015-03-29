### *static assert*
---
**概要**
プログラムの仕様、制約の表明としてassertが使われるが、このassertはランタイムでしかチェックすることが出来ず、エラーの場合にはプログラムのterminateが発生する。
コンパイル時に制約のチェックが出来るのであればコンパイルエラーとする事でバグを未然に防ぐことも可能になる。
これを実現する機能が static\_assert である。

構文を以下に示す。

`static_assert( constant-expression , 文字列リテラル );`

constant-expressionに指定した式はbool型に変換され、trueである場合には何も行わず、falseの場合にはコンパイルエラーとなり、文字列リテラルで指定した内容がエラー内容として出力される。constant-expressionとして指定可能な式は色々な条件があるが、constexprが指定された変数、関数を用いた条件文と考えれば良い。
定数式として認められるのであれば、constexprでなくても良い。

```c++
constexpr int f() {
    return 10;
}

int main(int argc, char* argv[]) {
    static_assert( f() == 10, "f() is not 10.");                    //OK

    constexpr int X = 100;

    static_assert( f() == X, "X don't equal f().");                 //error X don't equal f()

    static_assert( sizeof(int) == 8, "Integer size is not 8");      //OK "sizeof(int) == 8" is constant-expression

    return 0;
}
```

