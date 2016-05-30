## ライブラリ
C++14での標準ライブラリに対する大きな変更は特にない。主に小さな機能追加や、C++14での言語機能変更に伴う修正となっている。
実は他にも便利な機能(optional, dynarray)なども追加される予定であったが、C++14は機能修正のみであるべきだという主張によりドラフトから削除された。
本章では、C++14での機能追加と機能修正のうち、ポイントになりそうな変更点について解説する。

以下に列挙する内容について解説する。
 * [functorの利用簡単化](library/simplification_functor.md)
 * [exchange](library/excange.md)
 * [コンパイル時整数列](library/compile_time_int_seq.md)
 * [型によるタプルアクセス](library/access_tuple_by_type.md)
 * [make_unique](library/make_unique.md)
 * [Quoted Strings](library/quoted_strings.md)
 * [ライブラリ定義リテラル](library/user_def_literal.md)
 * [共有ロック](library/shared_lock.md)

上記で示した以外の変更は、C++のエキスパートが意識するような機能であるため、本書では省略する。

C++14でライブラリに追加された機能の一覧は以下のページを参照。

[libc++ C++1Y Status](http://libcxx.llvm.org/cxx1y_status.html)

