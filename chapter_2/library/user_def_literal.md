### *ライブラリ定義リテラル*
---
#### 概要
C++11で[ユーザ定義リテラル](chapter_1/core/user_defined_literal.md)機能が追加され、独自のsaffixリテラル記号を定義できるようになった。
C++14ではこの機能を用いて標準ライブラリで定義されている[chrono](chapter_1/librar/chrono.md), 複素数(complex), 文字列(std::string)を使いやすくするためのリテラルが定義された。

ライブラリ定義のユーザ定義リテラルはインラインネームスペースを用いて定義されており、以下のようになる。

```c++
namespace std{
inline namespace literals{
inline namespace string_literals{
basic_string<char> operator "" s(const char *str, size_t len);
basic_string<wchar_t> operator "" s(const wchar_t *str, size_t len);
basic_string<char16_t> operator "" s(const char16_t *str, size_t len);
basic_string<char32_t> operator "" s(const char32_t *str, size_t len);
}
inline namespacce chrono
}}
```

リテラルは::でスコープ指定せずに名前解決出来る必要があるため、usingディレクティブを用いる必要がある。
上記で示した例の場合では、以下のどれかを使用する。

```c++
using namespace std
using namespace std::literals;
using namespace std::literals::string_literals;
```

以下にライブラリに追加されたユーザ定義リテラルの一覧を示す。

| サフィックス | ネームスペース | 戻り値型 | 使用例 |
| -- | -- | -- |
| s | string_literals | std::string | "abc"s |
| s | string_literals | std::wstring | L"abc"s |
| s | string_literals | std::u16string | u"abc"s |
| s | string_literals | std::u32string | U"abc"s |
| h | chrono_literals | std::chrono::hours | 12h |
| min | chrono_literals | std::chrono::minutes | 12min |
| s | chrono_literals | std::chrono::seconds | 12s |
| ms | chrono_literals | std::chrono::milliseconds | 12ms |
| us | chrono_literals | std::chrono::microseconds | 12us |
| ns | chrono_literals | std::chrono::nanoseconds | 12ns |
| il | complex_literals | std::complex<long double> | -1.0il |
| i | complex_literals | std::complex<double> | -1.0i |
| if | complex_literals | std::complex<float> | -1.0if |

