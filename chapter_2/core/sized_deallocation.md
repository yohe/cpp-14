### *サイズ指定delete*
---
#### 概要
delete関数は、クラスメンバ関数と、グローバルスコープの２種類が存在する。なお、これ以外(グローバル以外のnamespace scope)で宣言されている場合は、ill-formedとなる。

C++11までのdelete関数は、クラスメンバ関数の場合のみ２番目の引数に`std::size_t`が指定可能なオーバーロードが定義可能となっていた。
C++14では、クラスメンバ関数のみ許可されていたサイズ指定を、グローバルスコープのdelete関数にも許可された。

追加されたdelete関数は以下のとおりである。

```c++
void operator delete(void*, std::size_t) noexcept;
void operator delete[](void*, std::size_t) noexcept;
```

引数を1つ持つdelete関数と引数を2つ持つdelete関数の両方が定義されている場合、後者のdelete関数が使用される。
サイズの指定は自動で行われる。

```c++
void operator new(std:size_t size) {
    return malloc(size);
}
void operator delete(void* ptr) noexcept{                       //#1
    std::cout << "delete 1" << std::endl;
    free(ptr);
}

void operator delete(void* ptr, std::size_t sz) noexcept{       //#2
    std::cout << "delete 2" << std::endl;
    free(ptr);
}

int main() {
    int* x = new int();
    delete x;               // call #2
}
```

