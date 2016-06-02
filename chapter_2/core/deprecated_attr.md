### *Deprecated attribute*
---
#### 概要
以前よりC++の複数のコンパイラでは、独自機能として非推奨である事を記述する事が出来ていた。
しかし、コンパイラの独自仕様で定義されたものであり、コンパイラごとに構文が異なっていた。
C++11にて、[attribute](chapter_1/core/attribute.md)が追加されたため、
統一化のためにC++14で非推奨を明示する属性(attribute)が定義された。

attribute listに対して、以下のどちらかを用いて指定する。

 * deprecated
 * deprecated("警告メッセージ")

この非推奨属性は、以下に示す宣言に対して指定可能である。
 | 指定可能な宣言 | 指定例 |
 | -- | -- |
 | クラス | class [[deprecated]] X; |
 | typedef | typedef int [[deprecated]] X; |
 | 変数 | int X [[deprecated]]; |
 | 非staticメンバ変数 | class X { int x [[deprecated]]; }; |
 | 関数/メンバ関数 | [[deprecated]] void func(); |
 | enum | enum [[deprecated]] E; enum class [[deprecated]] E; |
 | テンプレート特殊化 | template<> class [[deprecated]] X<int> {} |

また、deprecated指定無しで宣言のみされた名前は後でdeprecated付きで再宣言可能である。

```c++
void func();

[[deprecated]]
void func();        //OK

void func() {
}

int main() {
    return 0;
}

```

