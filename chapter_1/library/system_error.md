### *system error*
---
#### 概要
system errorとは、 OSなどのシステムコールや低レベルのAPIを使用するライブラリが、システムコールなどで発生したエラーを通知するための例外機能である。
簡単に言えば、errnoやperrorの機能をC++用に置き換えて、さらに拡張性を加えたものである。
C++11では、Iostreamや、スレッド関連で使用されている。

system errorは幾つかのクラスに分かれており、**error_category**、**error_code**、**error_condition**、**system_error**というクラスが存在する。

##### error_category
エラーコードの発生源や特定のエラーカテゴリを識別するためのベースクラスである。
基本的に既に定義されている一般カテゴリ, システムカテゴリを使用することになる。
また、ユーザが新たに定義することも可能でありその場合はerror_categoryクラスを継承する必要が有る。
一般カテゴリは、POSIXで定義された errno (ヘッダファイルcerrorに定義されているもの)に関連するエラーで使用される。
システムカテゴリは、OS毎に独立したエラーを示す場合に使用される。

これらのカテゴリはそれぞれ、以下の関数を用いて取得する。error_categoryは、参照でオブジェクトのやり取りする。
そのため、 これらの関数は常に同じオブジェクトを返却する。(つまり同じアドレスを持つ)
また、独自のerror_categoryを定義することで、プログラム毎の独自エラーコードとエラーメッセージの対応を定義することも可能であり、
その場合も同じカテゴリは同じアドレスを持つように定義すべきである。

 * const error_category& generic_category() noexcept;
 * const error_category& system_category() noexcept;

以下に、独自のerror_categoryを定義する場合の例を示す。

```c++
enum class my_error {
    no_error = 0,
    error = 1,
};

class MyErrorCategory : public std::error_category {
public:
    MyErrorCategory() : std::error_category() {}
    const char* name() const noexcept
    { return "my_error"; }

    std::string message(int ev) const
    { return msg.at(ev); }

private:
    std::map<int, std::string> msg = { {0,"no error"}, {1, "error occured"} };
};

static MyErrorCategory my_error_category;

const std::error_category& my_category() noexcept
{ return my_error_category; }

std::error_code make_error_code(my_error me) noexcept
{ return std::error_code(static_cast<int>(me), my_category()); }



int main() {
    std::error_code ec = make_error_code(my_error::no_error);
    std::cout << ec.message() << std::endl;
    ec = make_error_code(my_error::error);
    std::cout << ec.message() << std::endl;
    return 0;
}
```

##### error_code
環境依存のエラーコード値を保持するクラス。system_errorオブジェクトを構築する際に使用する。
error_codeクラスには、エラーコード値と、error_categoryオブジェクトを指定する。
このクラスを使用することで、エラーコード値に対応するエラーメッセージなどを取得することができる。

POSIX定義のエラーコードを使用する場合、`std::errc`に列挙子として定義されている。
このエラーコードを使用する場合、一般カテゴリのerror_categoryを指定する。

```c++
#include <system_error>
#include <iostream>

int main() {
    std::error_code ec { static_cast<int>(std::errc::no_such_file_or_directory),
                         std::generic_category() };
    std::cout << ec.value() << std::endl;               //2
    std::cout << ec.message() << std::endl;             //No such file or directory
    return 0;
}
```


##### error_condition
エラー状態を識別する環境非依存の値を保持するクラス。
error_conditionは、環境非依存のエラー値を保持するという点以外は、error_codeと同じである。

##### system_error
環境依存のエラー情報を表現する 例外(exception) である。環境依存のエラーを表現するため、error_code を保持する。
なお、コンストラクタ時に指定する "what_arg" 引数により what() 時の表現を変更することも可能である。
概ね `whart_arg + ": " + code.message()` のような表現となる。

```c++
#include <system_error>
#include <iostream>

int main() {
    std::error_code ec { 2, std::generic_category() };
    std::system_error exception(ec, "c++11");
    std::cout << exception.what() << std::endl;
    return 0;
```

```
c++11: No such file or directory
```
