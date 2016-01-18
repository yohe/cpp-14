### *文字列リテラル*
---
#### 概要
C++03では2種類の文字列リテラルが存在していた。通常のダブルクォーテーション(")で括る形式と、プレフィックスとしてLを指定する形式である。
これに対し、C++11では4種類の文字列リテラルが追加されている。

| シンタックス | セマンティクス | C++03/C++11 |
| -- | -- | -- |
|  "文字列..."  | 要素数 n の const char 配列 | C++03 and C++11 |
| L"文字列..." | 要素数 n の const wchar_t 配列 | C++03 and C++11 |
| u8"文字列..." | UTF-8にエンコード済み文字列 要素数 n の const char 配列 | C++11 |
| u"文字列..." | 要素数 n の const char16_t 配列<BR> | C++11 |
| U"文字列..." | 要素数 n の const char32_t 配列<BR> | C++11 |
| R"delimiter(文字列...)delimiter"<BR>※ delimiterは省略可能 | 要素数 n の配列(型は"R"の前に指定したencoding prefix [L, u8, u, U] に依存) | C++11 |
char16_t, char32_tは、それぞれUCS2の範囲の文字、UCS4の範囲の文字を扱うことができる。UCSとUnicodeはほぼ互換であるため、UTF-16, UTF-32にそれぞれ対応すると考えれば良い。
なお、UTF-16の場合はサロゲートペアを用いる場合がある。よってchar16_t型の文字列の場合、サロゲートペア形式を用いて複数の値(例えば2文字分)を1文字として表現する可能性もある。(UCS2は固定幅文字のみであり、UTF-16はマルチバイト文字も扱う。)
また、char16_t型文字列、char32_t型文字列は、Unicodeとの互換性を保つために、0x0 - 0x10FFFFまでのコード値のみで構成されなければならない。

※ 本書では、Unicodeで定義されているエンコード方式を示す場合UTFとして記載し、ISO/IEC 10646で規定されているコードポイント値を示す場合UCSとして記載する。



R"xxx"の文字列リテラル以外は、L"xxx"の場合と同様なイメージで使用できる。またエンコードに関しては変換ライブラリも提供されている。

```c++
#include <codecvt>

void func() {
    const char* utf8_string = u8"abcdefg";
    const wchar_t* w_string = L"abcdefg";
    const char16_t* utf16_string = u"abcdefg";
    const char32_t* utf32_string = U"abcdefg";

    std::cout << utf8_string << std::endl;
    std::wcout << w_string << std::endl;

    //UTF-32 -> UTF-8
    std::wstring_convert<std::codecvt_utf8<char32_t>, char32_t> cv;
    for(auto v : std::u32string(utf32_string)) {
        std::cout << cv.to_bytes(v);
    }

    //UTF-16 -> UTF-8
    std::wstring_convert<std::codecvt_utf8_utf16<char16_t>, char16_t> cv2;
    for(auto v : std::u16string(u16)) {
        std::cout << cv2.to_bytes(v);
    }
    std::cout << std::endl;
}
```

R"xxx"の文字列リテラルを Raw string literal と呼ぶ。これは、括弧内の文字列すべてをそのまま文字として認識する。改行やエスケープシーケンスもそのまま文字として認識される。
また、Raw string literal には、文字列の開始と終了を示す delimiter を指定することも可能である。これにより、)" もそのまま文字として認識させることもできる。

以下にサンプルを示す。

```c++
int main() {
    const char* raw = R"(a          //OK 改行も含む
bcd                                 //OK 改行後も ""で括る必要なし
   efg)";                           //OK
    std::cout << raw << std::endl;

    const char* raw_dlm = R"dlm(ab)"cdefg)dlm";     //OK )"以降も文字列として認識される
}
```
上記を実行した場合の結果を以下に示す。

```
a
bcd
   efg
ab)"cdefg
```
この実行結果が示すように、改行や空白も記述した通りにそのまま出力される。また、delimiterを指定することで )" を含む文字列も記述可能となる。
文字列リテラルと同様に文字リテラルも幾つか追加されている。いかにその種類を示す。

| シンタックス | セマンティクス | C++03/C++11 |
| -- | -- | -- |
|  '文字'  | const char 型| C++03 and C++11 |
| L'文字' | const wchar_t 型 | C++03 and C++11 |
| u'文字' | const char16_t 型<BR> UCSの16bit(UCS2)で表現可能な文字のみ(基本多言語面のみを扱う) | C++11 |
| U'文字' | const char32_t 型<BR> UCSの32bit(UCS4)で表間可能な文字のみ(UCSの全ての文字を扱う) | C++11 |


また、文字を Universal Character Set(UCS) のコードポイントで記述することも可能となった。

| シンタックス | セマンティクス | C++03/C++11 |
| -- | -- | -- |
| \uNNNN | ISO/IEC 10646 の 0000NNNN を示す | C++11 |
| \UNNNNNNNN | ISO/IEC 10646 の NNNNNNNN を示す | C++11 |


```c++
int main() {
    const char16_t* s = u"\u3024いう";      //OK あいう
    return 0;
}
```

