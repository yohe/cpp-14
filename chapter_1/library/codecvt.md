### *codecvt*
----
#### 概要
C++11の文字列リテラルの追加により、標準規格の中で様々な文字列のコーディング方式(UTF-8, UTF-16, UTF-32, UCS2, UCS4)が扱えるようになっており、
それに対応するためのコーディング変換機能も追加されている。ここでは、追加された機能について紹介する。

| 概要 | 名前 |
| -- | -- |
| UTF-8 と UCS2/UCS4 間の変換 | codecvt_utf8 |
| UTF-16 と UCS2/UCS4 間の変換 | codecvt_utf16 |
| UTF-8 と UTF-16 間の変換 | codecvt_utf8_utf16 |

上記で示した各機能は、ヘッダファイル *codecvt* をインクルードすることで利用可能である。
以下に、上記の各機能について説明する。

##### codecvt_utf8
マルチバイトシーケンスであるUTF-8と固定幅であるUCS2 or UCS4間の変換を提供する。

以下に **codecvt_utf8** の定義について示す。

```c++
namespace std {
template <class Elem, unsigned long Maxcode = 0x10ffff,
  codecvt_mode Mode = (codecvt_mode)0>
class codecvt_utf8 : public codecvt<Elem, char, mbstate_t> {
  // unspecified
};
}
```

UCS2 か UCS4 であるかの指定は、Elemに指定する型(char16_t, char32_t)に依存する。
Maxcodeとは対象とする文字空間の範囲である。UCSでは、0x0000 - 0x10ffffである。
また、Modeには処理する際の動作方式を指定する。Modeの定義を以下に示す。

```c++
namespace std {
enum codecvt_mode {
  consume_header = 4,       // 読み込み時にエンディアン情報を示すヘッダ(BOM)を処理する
  generate_header = 2,      // 出力時にエンディアン情報を示すヘッダ(BOM)を出力する
  little_endian = 1         // リトルエンディアンでマルチバイトシーケンスが生成されている
};
}
```

以下に簡単な例を示す。なお、動作環境はリトルエンディアンである。

```c++
#include <iostream>
#include <codecvt>

int main() {
    const char* utf8 = u8"あいうえお";
    std::cout << std::hex;
    for (auto c : std::string(utf8)) {
        unsigned char v = c;
        std::cout << (int)v << ",";
    }
    std::cout << std::endl;
    const char16_t* utf16 = u"あいうえお";
    std::cout << std::dec;
    for (auto c : std::u16string(utf16)) {
        std::cout << c << ",";
    }
    std::cout << std::endl;

    std::wstring_convert<std::codecvt_utf8<char16_t, 0x10ffff, (std::codecvt_mode)1>, char16_t> ws_cvt;
    std::u16string str = ws_cvt.from_bytes(std::string(utf8));
    for(auto& c : str) {
        unsigned int v = c;
        std::cout << v << ",";
    }
    return 0;
}
```

出力例を以下に示す。
```
UTF-8   : e3,81,82,e3,81,84,e3,81,86,e3,81,88,e3,81,8a,
UCS-2   : 12354,12356,12358,12360,12362,
Convert : 12354,12356,12358,12360,12362,
```

##### codecvt_utf16
マルチバイトシーケンスであるUTF-16と固定幅であるUCS2 or UCS4間の変換を提供する。

以下に **codecvt_utf16** の定義について示す。

```c++
namespace std {
template <class Elem, unsigned long Maxcode = 0x10ffff,
  codecvt_mode Mode = (codecvt_mode)0>
class codecvt_utf16 : public codecvt<Elem, char, mbstate_t> {
  // unspecified
};
}
```

UCS2 か UCS4 であるかの指定は、Elemに指定する型(char16_t, char32_t)に依存する。

以下に簡単な例を示す。なお、動作環境はリトルエンディアンである。

```c++
#include <iostream>
#include <codecvt>

int main() {
    const char32_t* ucs4 = U"\U00010384";
    std::cout << std::hex << "UCS-4   : " ;
    for (auto c : std::u32string(ucs4)) {
        int v = c;
        std::cout << (int)v << ",";
    }
    std::cout << std::endl;
    const char16_t* ucs2 = u"\U00010384";
    std::cout << std::hex << "UCS-2   : ";
    for (auto c : std::u16string(ucs2)) {
        std::cout << c << ",";
    }
    std::cout << std::endl;

    // ucs4 -> UTF-16
    std::wstring_convert<std::codecvt_utf16<char32_t, 0x10ffff, (std::codecvt_mode)1>, char32_t> ws_cvt;
    std::string str = ws_cvt.to_bytes(ucs4);
    std::cout << std::hex << "Convert : ";
    int count = 0;
    unsigned int v = 0;
    for(auto& c : str) {
        if( count % 2 == 0) {
            v = (0xFF&c);
        } else {
            v += (0xFF00&(c << 8));
            std::cout << v << ",";
            v = 0;
        }
        count++;
    }
    std::cout << std::endl;
}
```

出力例を以下に示す。

```
UCS-4   : 10384,
UCS-2   : d800,df84,
Convert : d800,df84,
```


##### codecvt_utf8_utf16
マルチバイトシーケンスであるUTF-8、およびUTF-16間の変換を提供する。

以下に **codecvt_utf8_utf16** の定義について示す。

```c++
namespace std {
template <class Elem, unsigned long Maxcode = 0x10ffff,
  codecvt_mode Mode = (codecvt_mode)0>
class codecvt_utf8_utf16 : public codecvt<Elem, char, mbstate_t> {
  // unspecified
};
}
```

以下に簡単な例を示す。なお、動作環境はリトルエンディアンである。
```c++
#include <iostream>
#include <codecvt>

int main() {
    std::cout << std::endl;
    const char16_t* ucs2 = u"\U00010384";
    std::cout << std::hex << "UCS-2   : ";
    for (auto c : std::u16string(ucs2)) {
        std::cout << c << ",";
    }
    std::cout << std::endl;

    // ucs2 -> UTF-8
    std::wstring_convert<std::codecvt_utf8_utf16<char16_t, 0x10ffff, (std::codecvt_mode)1>, char16_t> ws_cvt;
    std::string str = ws_cvt.to_bytes(ucs2);
    std::cout << std::hex << "Convert : ";
    for(auto& c : str) {
        unsigned int v = c;
        std::cout << (0xFF&v) << ",";
    }
    std::cout << std::endl;
}
```

出力例を以下に示す。
```

UCS-2   : d800,df84,
Convert : f0,90,8e,84,
```
