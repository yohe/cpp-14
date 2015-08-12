### *noexcept*
---
#### 概要
C++03のthrow(...)による例外指定は、C++11で非推奨機能となった。 その代わり、C++11では例外を送出する可能性があるか、送出しないかのみを指定する。
ただ、やはりC++03と同様に例外指定は言語による強制力は無いため、結局のところ欠陥のある機能である。
以下のように noexcept キーワードを用いて記述する。関数宣言時、関数定義時で指定内容が同じ意味でなければならない。

`void func() noexcept;`

関数funcは例外を送出しない。

`void func() noexcept (定数式);`

 関数funcは定数式が true である場合、例外を送出しない。逆にfalseである場合は例外を送出する可能性がある。

noexcept による例外指定が追加されると共に、与えた式が例外を送出する可能性があるかないかを判定する noexcept オペレータが追加された。
このオペレータは、例外を送出する可能性がある式を渡した場合に false を返却し、例外を送出しない式であれば true を返却する。
またnoexceptオペレータはコンパイル時に評価されるのみであり、与えられた式は実行時に評価されることはなく、動作に影響を与えない。
これを組み合わせることにより、ある関数が例外を送出する可能性があるかどうかで関数宣言の例外指定を切り替えることも可能である。
noexcept オペレータは以下のような構文となる。

`noexcept (定数式)`

```c++
void non_throw() noexcept {}            //OK 例外を送出しない

void throwfunc() noexcept(false) {}     //OK 例外を送出する可能性がある

constexpr bool trueFunc() {
    return true;
}
constexpr bool falseFunc() {
    return false;
}

void non_throw2() noexcept( truefunc() ) {}     //OK truefunc() -> true
void throwfunc2() noexcept( falsefunc() ) {}    //OK falsefunc() -> false

void non_throw3() noexcept( noexcept( non_throw() ) ) {}        //OK noexcept ( non_throw() ) -> true
void throwfunc3() noexcept( noexcept( throwfunc() ) ) {}        //OK noexcept ( throwfunc() ) -> false

int main() {
    
    if ( noexcept( non_throw() ) ) {
        std::cout << "noexcept(true)" << std::endl;
    }

    if ( noexcept( throwfunc() ) ) {
        std::cout << "noexcept(false)" << std::endl;
    }

    int i=0;
    if ( noexcept( i++==0 ) ) {             //OK i==0, i++は実行時に評価されることはない
        std::cout << "i==0" << std::endl;
    }
    assert( i==0 );     //OK i==0
    return 0;
}
```

