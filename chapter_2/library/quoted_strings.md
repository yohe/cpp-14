### *Quoted Strings*
---
#### 概要
C++の文字列操作において、""で囲まれた文字列を１つのエレメントとして扱うような機能を直接サポートしていない。
また、文字列内のスペースについても注意して扱う必要がある。

例えば、""や、文字列内のスペースを意識しない場合には以下のようになる。
```c++
#include <sstream>
#include <string>

int main() {
    std::string original = "\"foolish me\"";
    std::string round_trip;

    std::stringstream ss;
    ss << original;
    ss >> round_trip;                   //期待="foolish me", 実際="foolish

    assert(original == round_trip);     //assert!
    return 0;
}
```

C++14で、""で囲まれたことを示すマニピュレータ`quoted`が追加された。
なお、デフォルトの動作はダブルクォートであり、シングルクォートに変更することも可能である。
以下に示すquotedの  定義を見ればわかるように、デリミタとエスケープ記号も指定することが可能である。

```c++
template <class charT>
unspecified quoted(const charT* s, charT delim=charT(’"’), charT escape=charT(’\\’));

template <class charT, class traits, class Allocator>
unspecified quoted(const basic_string<charT, traits, Allocator>& s,
                   charT delim=charT(’"’), charT escape=charT(’\\’));

template <class charT, class traits, class Allocator>
unspecified quoted(basic_string<charT, traits, Allocator>& s,
                   charT delim=charT(’"’), charT escape=charT(’\\’));
```

以下にサンプルを示す。
```c++
#include <sstream>
#include <iomanip>
#include <string>

int main() {
    std::string original  = "\"foolish me\"";
    std::string round_trip ;

    std::stringstream ss;
    ss << original;
    ss >> round_trip;

    std::cout << original << std::endl;         // "foolish me"
    std::cout << round_trip << std::endl;       // "fooish

    ss.str("");
    round_trip = "";
    ss << std::quoted(original);
    ss >> std::quoted(round_trip);

    std::cout << round_trip << std::endl;       // "foolish me"
}
```

