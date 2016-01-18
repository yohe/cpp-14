### *スマートポインタ*
---
#### 概要
スマートポインタとは、メモリの動的確保/解放をクラス内に隠蔽し、メモリの解放漏れや2重解放などを回避する賢いポインタである。
また、スマートポインタは生ポインタと同じ操作性を提供するように設計されており、`*p`や、`p->`によりポインタがさすオブジェクトを参照できる。

メモリ管理を考える上で、所有権の考えが非常に重要である。所有権を持つということは、解放の義務を持つことでもある。
所有権を持っている間は、メモリの解放が行われることは無い。
所有権を複数で共有するスマートポインタもあるが、所有権を誰も持たなくなるまではメモリの解放は行われ無い。

C++03にも、auto_ptrと呼ばれるスマートポインタが定義されていたが、様々な問題からC++11では非推奨となっている。
その代わりに、C++11には大きく分けて2つのスマートポインタが導入された。
これらのスマートポインタは、ヘッダファイル**memory**をインクルードすることで使用可能となる。

 * unique_ptr (ポインタの所有権を常に一人が持つ)
 * shared_ptr (ポインタの所有権を複数人で共有する)

スマートポインタのもう一つの重要な機能としてdeleterが存在する。本書ではdeleterに関する解説は省略する。
deleterとは、ポインタを解放する際の振る舞いをカスタマイズするものである。特殊なメモリ領域の管理をし無い限りは気にする必要は無い。

##### unique_ptr
unique_ptrとは、所有権を常に一人だけが持つポインタである。よってコピー(所有権の複製)は不可で、ムーブ(所有権の移譲)のみ許可されている。
ムーブについては[r-value reference](../core/r_value_ref.md)を参照。

unique_ptrの構築は、管理対象のポインタを渡すだけである。C++11時点では、newにより確保したポインタを渡すため、コード上からnewを消し去ることはできない。
C++14ではこの点が改善されておりヘルパ関数`std::make_unique`を呼び出すことでunique_ptrを構築できるようになっている。
また、unique_ptrは、動的な長さの配列も管理することができる。こちらは通常版に追加して、'operator[]'によるアクセス機能を提供している。

```c++
#include <memory>

class Person {
public:
    Person() : age_(0), male_(true) {}
    Person(int age, bool male) : age_(age), male_(male) {}

    int age() const { return age_; }
    bool is_male() const { return male_; }

private:
    int age_;
    bool male_;
};

int main() {
    std::unique_ptr<Person> person = 
        std::unique_ptr<Person>(new Person(20, true));              //OK newと同時に渡す

    std::cout << person->age() << std::endl;
    std::cout << (*person).is_male() << std::endl;

    Person* p = new Person(30, false);
    std::unique_ptr<Person> person_2 = std::unique_ptr<Person>(p);  //OK new済みオブジェクトへのポインタを渡す

    std::unique_ptr<Person[]> person_arr = 
        std::unique_ptr<Person>(new Person[5]);                     //OK new[]と同時に渡す
    person_arr[1].is_male();                                        //OK 2個目の要素へのアクセス

    return 0;
}
```

unique_ptrは、管理しているポインタを管理対象からはずしたり、別のポインタを登録し直す事も出来る。登録し直す場合、当然古いポインタは解放される。
管理対象から外す場合は、ユーザがメモリの管理を行わなければならない。

```c++
#include <memory>

int main() {
    std::unique_ptr<int> i = std::unique_ptr<int>(new int());

    int* p = i.release();                           //OK ポインタを管理対象から外す

    assert(i.get() == nullptr);                     //OK 管理対象から外されると nullptr を持つ

    delete p;

    i.reset(new int());                             //OK ポインタの再登録
    i.reset(new int());                             //OK ポインタの再登録, 古いポインタは削除される

    return 0;
}

```

##### shared_ptr
shared_ptrは、共有可能な所有権の提供を行う。参照カウント方式を採用しており、最後に所有権を持っている所有者が解放の義務を遂行する。
このため、循環参照になっていると解放が行われず、メモリリークとなってしまう。

これを解決するために`weak_ptr`がある。これは、shared_ptrから構築出来る所有権を持たないポインタである。
所有権を持たないため、参照先のポインタが解放される可能性があるが、weak_ptrには、親となるshared_ptrが有効かどうかを判定する機能がある。
そして、有効な場合にweak_ptrからshared_ptrへの昇格を行うことでメモリアクセス出来るようになる。

上記以外は、基本的な使用方法は、unique_ptrと同じである。
ただし、unique_ptr とは異なり、コピーが可能となっている。コピーを行うことは、所有者が増えることを意味する。

```c++
#include <memory>
#include <iostream>
#include <cassert>

struct Parent {
    std::shared_ptr<Child> child;
    Parent() { std::cout << "Parent destructor" << std::endl;
};

struct Child {
    std::weak_ptr<Parent> parent;                               //循環参照をweak_ptrで回避する
    Parent() { std::cout << "Child destructor" << std::endl;
};

int main() {
    std::weak_ptr<int> wp;
    {
        std::shared_ptr<int> sp = std::shared_ptr<int>(new int(10));
        std::cout << *sp << std::endl;
        wp = sp;
        if (! wp.expire() ) {                               //OK 親のshared_ptrが有効かどうか判定
            std::cout << sp.use_count() << std::endl;       //OK 1 (所有者は1)
            std::shared_ptr<int> sp2 = wp.lock();           //OK shared_ptrへの昇格
            std::cout << sp.use_count() << std::endl;       //OK 2 (所有者は2)
        }
    }
    assert( wp.expire() );

    std::shared_ptr<Parent> p = std::shared_ptr<Parent>(new Parent);
    std::shared_ptr<Child> c = std::shared_ptr<Child>(new Child);
    p->child = c;                                                       //OK shared_ptrのコピー
    c->parent = p;
    return 0;
}
```

なお、shared_ptrには、構築関数make_shared()が提供されている。これを用いることで、オペレータnew を記述する必要がなくなる。
unique_ptrには、C++11時点では提供されていないが、C++14で追加された。
