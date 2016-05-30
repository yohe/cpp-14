### *型によるタプルアクセス*
---
#### 概要
C++11で追加されたタプル`std::tuple`の各要素へアクセスするために、`std::get<N>`(Nは整数)が提供されていた。
このget関数に対して型指定でもタプルの要素へアクセスする機能が追加された。
型指定でタプルへアクセスする場合には、指定した型がタプル内に複数存在する場合は、ill-formedとなる。

```c++
#include <string>
#include <tuple>

int main() {
    std::tuple<std::string, std::string, int> t("foo", "bar", 7);
    int i = std::get<int>(t);               //OK i == 7
    int j = std::get<2>(t);                 //OK j == 7
    std::string j = std::get<string>(t);    //error
}
```

