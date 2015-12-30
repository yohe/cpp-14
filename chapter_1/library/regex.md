### *regex*
---
#### 概要
C++11には正規表現による文字列操作ライブラリが追加された。正規表現で扱える文字列操作アルゴリズムは、文字列照合、文字列検索、文字列置換の３種類がある
文字列を扱う場合には文字の型を意識して使用する必要がある。C++11ではchar, wchar_tの２種類について動作する事が保証されているが、
char16_t, char32_tを扱える事を標準規格では求めていないため、扱う場合は注意が必要である。

char, wchar_tを扱う場合、簡単に利用するために以下のようなtypedefが定義されている。

| 定義 | typedef |
| -- | -- |
| basic_regex<char> | regex |
| basic_regex<wchar_t> | wregex |
| sub_match<const char*> | csub_match |
| sub_match<const char*> | csub_match |
| sub_match<const wchar_t*> | wcsub_match |
| sub_match<string::const_iterator> | ssub_match |
| sub_match<wstring::const_iterator> | wssub_match |
| match_result<const char*> | cmatch |
| match_result<const wchar_t*> | wcmatch |
| match_result<string::const_iterator> | smatch |
| match_result<wstring::const_iterator> | wsmatch |

他にもregex_iterator, regex_token_iterator等も定義されている。

基本的な正規表現ライブラリの利用方法は、上記の[w]regexに対して正規表現の式を与え、対象文字列と[w]regexを文字列操作アルゴリズムに渡し、match_resultを得るという流れになる。
正規表現が適切でない場合、 `regex_error` の例外が[w]regexのコンストラクタからthrowされる。

```c++
#include <regex>
#include <iostream>

int main() {
    try {
        std::regex re("[0-9+");
        std::string str("12345");
        regex_match (str, re);
    } catch (std::exception& e) {
        std::cout << e.what() << std::endl;
    }
}
```

また、正規表現を扱う上で幾つかのsyntaxオプションが定義されている。以下にオプション定義と効果について示す。

| 定義 | 効果 |
| -- | -- |
| std::regex_constants::icase | 大文字小文字の違いを無視する |
| std::regex_constants::nosubs | サブ式は非サブ式として扱われ、またサブ式が一致したものをmatch_resultに格納しない |
| std::regex_constants::optimize | 高速化フラグ |
| std::regex_constants::collate | [a-z]のような範囲指定がロケールを意識する|
| std::regex_constants::ECMAScript | ECMAScriptのECMA-262の文法として扱う |
| std::regex_constants::basic | POSIXの文法として扱う |
| std::regex_constants::extended | POSIXの拡張文法として扱う |
| std::regex_constants::awk | POSIXのawk文法として扱う |
| std::regex_constants::grep | POSIXのgrep文法として扱う |
| std::regex_constants::egrep | POSIXのegrep(grep -E)文法として扱う |

以下に、各文字列操作アルゴリズムの使用方法を示す。

##### 文字列照合 (regex_match)
文字列照合は、対象文字列が指定した正規表現と完全に一致するかどうか(true or false)を判定する。一部でも適合しない場合は不一致として判定される。
以下に regex_match の使用例を示す。

```c++
#include <regex>
#include <iostream>

int main() {
    std::regex re("[a-z]*[0-9]*");
    std::string target("abcdefg012345");
    if ( std::regex_match(target, re) ) {
        std::cout << "matched" << std::endl;
    }
}
```

正規表現のアルゴリズムに渡す対象文字列は、イテレータ、ポインタ、[w]stringの３種類が存在している。
また、処理結果を match_result として受け取ることも可能である。match_resultは、正規表現に適合した文字列の集合を表現する。
サブ式が含まれていた場合、正規表現に適合した全体の文字列、サブ式に適合した文字列を持つ。
集合の0番目が全体文字列、1〜N番目にサブ式の適合文字列が入る。

```c++
#include <regex>
#include <iostream>

int main() {
    std::regex re("([a-z]*)([0-9]*)");
    std::string target("abcdefg012345");
    std::smatch result;

    if ( std::regex_match(target.cbegin(), target.cend(), result, re) ) {
        std::cout << "matched" << std::endl;
        std::cout << result[0] << std::endl;        // 添字によるアクセスが可能
    }


    for (auto& v : result) {                        // ループで各要素にアクセス
        std::cout << v << std::endl;
    }

    for (int i=0; i<result.size(); i++) {
        std::cout << result.str(0) << std::endl;
    }
}
```

##### 文字列検索 (regex_search)
文字列検索は、対象文字列が指定した正規表現と一致する文字列が存在するかどうか(true or false)を判定する。一部でも適合しない場合は不一致として判定される。
文字列検索についても文字列照合と同様にmatch_resultで処理結果を受け取ることができる。
文字列検索は部分文字列の検索に使用することになるため、文字列の先頭から正規表現に適合した部分文字列までのprefix, 部分文字列から文字列の終端までのsuffixも得ることができる。
以下に regex_search の使用例を示す。

```c++
#include <regex>
#include <iostream>

int main() {
    std::regex re("\\.");                       // "\."では通常のエスケープシーケンスとして判断されるため、"\\."を用いる
    std::string target("abc.efg");
    if ( std::regex_match(target.cbegin(), target.cend(), result, re) ) {
        std::cout << "matched" << std::endl;
    }
    std::cout << result.prefix() << std::endl;  // abc
    std::cout << result.suffix() << std::endl;  // efg
}
```

##### 文字列置換 (regex_replace)
文字列置換は、対象文字列に対して正規表現にマッチした部分文字列を指定したformat文字列で置き換える。
以下に regex_replace の使用例を示す。

```c++
#include <regex>
#include <iostream>

int main() {
    std::regex re("[0-9]");
    std::string target("a0b1c2d3");
    std::string result = std::regex_replace(target, re, "*");

    std::cout << result << std::endl;       // a*b*c*d*
}
```

指定するformat文字列はデフォルトではECMAScriptのシンボルを扱うことが可能である。この動作はmatch_flag_typeによって変更可能である。
以下に指定可能なフラグを示す。

| 定義 | 効果 |
| -- | -- |
| std::regex_constants::format_default | ECMAScriptのルールに従って結果文字列を作成する |
| std::regex_constants::format_sed | sedのルールに従って結果文字列を作成する |
| std::regex_constants::format_no_copy | マッチした部分文字列以外は結果文字列に含めない |
| std::regex_constants::format_first_only | 最初にマッチした部分文字列のみを置換する |
※ここではformatに関係するフラグのみを記載している。マッチ方針に関するフラグは未記載

```c++
#include <regex>
#include <iostream>

int main() {
    std::regex re("[0-9]");
    std::string target("a0b1c2d3");
    using namespace std::regex_constants;
    match_flag_type flag;
    flag = format_no_copy;
    std::string result = std::regex_replace(target, re, "matched = $&\n", flag);
    std::cout << result << std::endl;
}
```

上記の実行結果は以下のようになる。

```
matched = 0
matched = 1
matched = 2
matched = 3

```
