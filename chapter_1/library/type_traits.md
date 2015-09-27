### *type traits*
----
#### 概要
ライブラリtype_traitsは、型特性を判定、操作、取得するユーティリティ機能を提供するライブラリである。
メタプログラミングや、ライブラリ作成などの再利用性の高い部品を作成する際にお世話になる機能だろう。

このライブラリに定義されている機能を全てすると膨大に成ってしまうので、ここでは簡単な説明のみとする。
詳細を知りたい場合は、参考サイトやC++標準規格を参照してほしい。


##### ヘルパクラス

| 定義名 | 概要 |
| -- | -- |
| integral_constant | 定数を表現 |
| true_type | trueを表現 |
| false_type | falseを表現 |

##### 基本的な型判定

| 定義名 | 概要 |
| -- | -- |
| is_void | void型であるかを判定 |
| is_integral | 整数型であるかを判定 |
| is_floating_point | 浮動小数点型であるかを判定 |
| is_array | 配列型であるかを判定 |
| is_pointer | ポインタ型であるかを判定 |
| is_lvalue_reference | 左辺値参照であるかを判定 |
| is_rvalue_reference | 右辺値参照であるかを判定 |
| is_member_object_pointer | データメンバへのポインタであるかを判定 |
| is_member_function_pointer | メンバ関数へのポインタであるかを判定 |
| is_enum | 列挙型であるかを判定 |
| is_union | 共有型であるかを判定 |
| is_class | クラス型であるかを判定 |
| is_function | 関数であるかを判定 |

##### 複合型の判定

| 定義名 | 概要 |
| -- | -- |
| is_reference | 参照であるかを判定 |
| is_arithmetic | 算術型であるかを判定 |
| is_fundamental | 基礎型であるかを判定 |
| is_object | オブジェクトであるかを判定 |
| is_scalar | スカラ型であるかを判定 |
| is_compound | 混合型であるかを判定 |
| is_member_pointer | メンバへのポインタであるかを判定 |

##### 型属性の判定

| 定義名 | 概要 |
| -- | -- |
| is_const | constであるかを判定 |
| is_volatile | volatileであるかを判定 |
| is_trivial | トリビアルな型であるかを判定 |
| is_trivial_copyable | トリビアルコピー可能な型であるかを判定 |
| is_standard_layout | 標準レイアウトであるかを判定 |
| is_pod | POD型であるかを判定 |
| is_literal_type | リテラル型であるかを判定 |
| is_empty | 空のクラスであるかを判定 |
| is_polymorphic | ポリモーフィックな型であるかを判定 |
| is_abstract | 抽象型であるかを判定 |
| is_signed | 符号付型であるかを判定 |
| is_unsigned | 符号なし型であるかを判定 |
| is_constructible | 構築可能な型であるかを判定 |
| is_default_constructible | デフォルト構築可能な型であるかを判定 |
| is_copy_constructible | コピー構築可能な型であるかを判定 |
| is_move_constructible | ムーブ構築可能な型であるかを判定 |
| is_assignable | 代入可能な型であるかを判定 |
| is_copy_assignable | コピー代入可能な型であるかを判定 |
| is_move_assignable | ムーブ代入可能な型であるかを判定 |
| is_destructible | 破棄可能な型であるかを判定 |
| is_trivially_constructible | トリビアルに構築可能な型であるかを判定 |
| is_trivially_default_constructible | トリビアルにデフォルト構築可能な型であるかを判定 |
| is_trivially_copy_constructible | トリビアルにコピー構築可能な型であるかを判定 |
| is_trivially_move_constructible | トリビアルにムーブ構築可能な型であるかを判定 |
| is_trivially_assignable | トリビアルに代入可能な型であるかを判定 |
| is_trivially_copy_assignable | トリビアルにコピー代入可能な型であるかを判定 |
| is_trivially_move_assignable | トリビアルにムーブ代入可能な型であるかを判定 |
| is_trivially_destructible | トリビアルに破棄可能な型であるかを判定 |
| is_nothrow_constructible | 無例外指定、かつ構築可能な型であるかを判定 |
| is_nothrow_default_constructible | 無例外指定、かつデフォルト構築可能な型であるかを判定 |
| is_nothrow_copy_constructible | 無例外指定、かつコピー構築可能な型であるかを判定 |
| is_nothrow_move_constructible | 無例外指定、かつムーブ構築可能な型であるかを判定 |
| is_nothrow_destructible | 無例外指定、かつ破棄可能な型であるかを判定 |
| has_virtual_destructor | 仮想デストラクタを持つか判定 |

##### 型属性のクエリ

| 定義名 | 概要 |
| -- | -- |
| alignment_of | アライメントを調べる |
| rank | 配列の次元数を調べる |
| extent | 配列のI次元目の要素数を調べる |

##### 型の関係

| 定義名 | 概要 |
| -- | -- |
| is_same | ２つの型が同じか判定 |
| is_base_of | DeriveがBaseを継承しているか判定 |
| is_convertible | FromからToへ変換可能か判定 |

##### 型変換

| 定義名 | 概要 |
| -- | -- |
| remove_const | const性の除去 |
| remove_volatile | volatile性の除去 |
| remove_cv | const, volatile性の除去 |
| add_const | const性の追加 |
| add_volatile | volatile性の追加 |
| add_cv | const, volatile性の追加 |
| remove_reference | 参照の除去 |
| add_lvalue_reference | 左辺値参照の追加 |
| add_rvalue_reference | 右辺値参照の追加 |
| make_signed | 符号付整数型への変換 |
| make_unsigned | 符号なし整数型への変換 |
| remove_extent | 次元数を一つ除去 |
| remove_all_extent | 次元数を全て除去 |
| remove_pointer | ポインタの除去 |
| add_pointer | ポインタの追加 |

##### その他の変換

| 定義名 | 概要 |
| -- | -- |
| aligned_storage | アライメント済みストレージ作成 |
| aligned_union | アライメント済みunion作成 |
| decay | ? |
| enable_if | 条件が真の場合に型を定義 |
| conditional | 条件に応じて型を切り替える |
| common_type | 全ての型の共通型を定義 |
| underlying_type | Enumの基盤型を取得 |
| result_of | 呼び出し可能な型の戻り値型を取得 |

