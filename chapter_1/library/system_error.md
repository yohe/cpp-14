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

これらのカテゴリはそれぞれ、以下の関数を用いて取得する。この関数は常に同じオブジェクトを返却する。(つまり同じアドレスを持つ)

 * const error_category& generic_category() noexcept;
 * const error_category& system_category() noexcept;


##### error_code
環境依存のエラーコード値を保持するクラス。system_errorオブジェクトを構築する際にerror_codeクラスを渡すことになる。
error_codeクラスには、エラーコード値と、error_categoryオブジェクトを指定する。
このクラスを使用することで、エラーコード値に対応するエラーメッセージなどを取得することができる。
エラーコード値は、`std::errc`に列挙子として定義されているので、そちらを使用するのがよい。

```c++
#include <system_error>
#include <iostream>

int main() {
    std::error_code ec { 2, std::generic_category() };
    std::cout << ec.value() << std::endl;               //2
    std::cout << ec.message() << std::endl;             //No such file or directory
    return 0;
}
```


##### error_condition
エラー状態を識別する環境非依存の値を保持するクラス。
TBE

##### system_error
TBE

