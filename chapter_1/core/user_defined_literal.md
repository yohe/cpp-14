### *ユーザ定義リテラル*
---
#### 概要
C++11では、リテラルをユーザが定義できるようになった。これをユーザ定義リテラルと呼ぶ。
ユーザ定義リテラルは、値(整数、浮動小数点、文字列、文字)に対して変換などの任意の処理を施すsuffixを定義する。
ユーザ定義リテラルを使用する際には、"::"によるスコープ指定はできないため、使用する場所での unqualified name lookup で呼び出されるユーザ定義リテラルに限られる。
そして、ユーザ定義リテラルで使用できるサフィックス名は、必ずアンダースコアで始まらないといけない。アンダースコア無しは、標準で予約されている。
※ 基本的にアンダースコアで始まる名前は標準で予約されているが、ユーザ定義リテラルのみアンダースコア無しが予約されている。

以下に構文を示す。

`result_type operator"" suffix_name (parameter list)`

パラメータリストの形式は、値の種別毎に定義可能な形式が決められている。

種別毎に形式を示す。

##### 整数
* result_type operator"" suffix_name (unsigned long long);
* result_type operator"" suffix_name (const char*);
* result_type operator"" suffix_name<'c1', 'c2', ... , 'ck'> ();

##### 浮動小数点
* result_type operator"" suffix_name (long double);
* result_type operator"" suffix_name (const char*);
* result_type operator"" suffix_name<'c1', 'c2', ... , 'ck'> ();

##### 文字列
* result_type operator"" suffix_name (const char*, size_t);
* result_type operator"" suffix_name (const wchar_t*, size_t);
* result_type operator"" suffix_name (const char16_t*, size_t);
result_type operator"" suffix_name (const char32_t*, size_t);
※ size_t には、null終端を抜いた文字列の長さが与えられる

##### 文字
* result_type operator"" suffix_name (char);
* result_type operator"" suffix_name (wchar_t);
* result_type operator"" suffix_name (char16_t);
result_type operator"" suffix_name (char32_t);

以下に使用例を示す。

```c++
std::string  operator"" _to_string(const char* number) {
    return std::string(number);
}

std::complex<double> operator"" _i(long double imaginary) {
    return std::complex<double>(0.0, imaginary);
}

int operator"" _to_int(const char* num, size_t len) {
    std::string str(num);
    int i;
    
}

namespace ns {
    std::string operator"" _i(const char* str) {
        return std::string(str);
    }
    void nsFunc() {
        std::string str = 1.0_i;                //OK ns::_i が呼び出される
    }
};

int main() {
    std::string number = 123_to_string;         //OK
    std::complex<double> c = 1.0 + 2.0_i;       //OK ::_i が呼び出される

    ns::nsFunc();
}
```
