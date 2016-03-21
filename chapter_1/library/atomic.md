### *atomic*
---
#### 概要
atomicとは、アトミック操作を実現するテンプレートクラス*atomic*を提供するライブラリである。atomic型を使用することでロックフリーなプログラムを作成可能となる。
atomic型を適切に使用するには、メモリモデル、およびフェンス(メモリバリア)についての理解が必要になる。

##### メモリモデル
メモリモデルを説明するのは非常に難しいため、メモリモデルの一部のみを扱う。そして、メモリバリアとその必要性について説明し、atomicでの表現方法を示すことにする。
本書で記述するメモリモデルとは、メモリ一貫性モデルを指す。メモリ一貫性モデルの一般的な内容は、[consistency model](https://en.wikipedia.org/wiki/Consistency_model)を参照してほしい。

メモリ一貫性モデルとは、マルチスレッドで動作するプログラム上の変数に対するアクセス(読み/書き)がどのような順序で実行されるかのルールを定義したものである。
メモリ一貫性モデルにおいて一番基本的なモデルが逐次一貫性モデルである。

> 逐次一貫性モデルとは、どのような実行結果も、すべてのプロセッサがある順序で逐次的に実行した結果と等しく、
> かつ、個々のプロセッサの処理順序がプログラムで指定された通りであること
>
> Wikipedia引用

上記の内容は当たり前のように感じられるかもしれないが、これを満たさないメモリ一貫性モデルも存在する。
例えば、Total Store Order(TSO), Relaxed consistency model(XC)などがある。これらはメモリアクセス命令をプログラム順とは異なる順で発行すること許容したメモリモデルである。分かりやすい例を挙げるならば、あるメモリの読み込み命令を実行する際に、そのメモリのデータがキャッシュ上にない、もしくは古いデータの場合にはメモリから読み込むまで待ち時間が発生する。この時に、後続する関連のないメモリアクセス命令であれば実行することが可能であるため、実行順を入れ替えることで高速化させることがある。

このようなプログラム順とメモリアクセス順がことなる順で実行されることをアウトオブオーダ実行と呼ぶ。なお、プログラム順のことをプログラムオーダと呼び、メモリアクセス順をメモリオーダと呼ぶ。
※ このアウトオブオーダはプログラムの最適化によっても引き起こされる可能性がある。


アウトオブオーダ実行について考えてみる。例えば、以下のコードを実行した場合、v1がどのような値を持つか考える。
なお、データ競合(data race)は発生しないものとする。

```c++
#include <thread>
#include <iostream>

int A = 0;
int B = 0;
int C = 0;

int v1 = 0;
int v2 = 0;
int v3 = 0;
int v4 = 0;

void thread1() {
    A = 1;          //i1
    v1 = A;         //i2
    v2 = B;         //i3
}

void thread2() {
    B = 1;          //j1
    v3 = B;         //j2
    v4 = A;         //j3
}


int main() {
    int i=0;
    while(i < 100000) {
        std::thread t1(thread1);
        std::thread t2(thread2);

        t1.join();
        t2.join();
        if (v2 == 0 && v4 == 0) {
            std::cout << "v1 : " << v1 << std::endl;
            std::cout << "v2 : " << v2 << std::endl;
            std::cout << "v3 : " << v3 << std::endl;
            std::cout << "v4 : " << v4 << std::endl;
        }
        A=B=C=0;
        v1=v2=v3=v4=0;
        i++;
    }

    return 0;
}
```

このプログラムをマルチコア環境で実行した時、v2 == v4 == 0 となる可能性がある。プログラム順を意識する限り、v2,v4がそれぞれ0となるように命令を一列に並べることは、不可能である。この現象は、あるCPU-Aで実行された書き込み処理が、CPU-Bから観測できるタイミングがプログラム順と異なる順で反映されることにより発生する。
つまりアウトオブオーダが発生することで引き起こされる。
アウトオブオーダーはプログラムの最終的な動作に影響がないように行われるが、これは実行スレッド自身に対してのみ考慮される。
従って、複数スレッドで動作するプログラムの場合、アウトオブオーダは予期しない動作を引き起こす可能性がある。


さて、このように逐次一貫性をハードウェアレベルで保証できないケースがある。そのため、ハードウェア側で順序性を保証する機能を提供している。それがフェンス(メモリバリア)である。C++におけるatomic操作はatomicな変数アクセスに加え、このフェンス機能を提供する。

C++ではatomic操作を行うにあたり、atomicでない変数のメモリオーダがどのように振る舞うかを指定することができる。
メモリオーダを考えるとき、以下のような分類で考えることができる。
 * Load -> Load
 * Load -> Store
 * Store -> Load
 * Store -> Store

上記は、矢印の左のメモリ操作の後に、右のメモリ操作が行われるメモリ操作順を示している。逐次一貫性モデルの場合、メモリ操作順はプログラム順と同じであることは前述した。しかし、それ以外のモデルの場合には同じ順に行われない可能性がある。つまり左のメモリ操作と右のメモリ操作が逆転する場合である。この逆転を防ぐためにフェンスを用いる。C++におけるフェンスは以下の種類に分類されている。

| フェンス | 保証する動作 |
| -- | -- |
| Acquire fence | Load -> Load, Load -> Store |
| Release fence | Load -> Store, Store -> Store |

Acquire fenceとは、フェンスに続く任意のメモリ操作が必ずフェンスの後に実行されることを示す。また、Acquire fenceはLoad操作に指定可能である。
Release fenceとは、フェンスの前の任意のメモリ操作が必ずフェンスの前に実行されることを示す。また、Release fenceはStore操作に指定可能である。

上記の表を見ると、Store -> Load 操作に対する保証がないことがわかる。つまり、Acquire fence, Release fenceだけでは全てのメモリ操作の組み合わせに対する順序保証を提供できない。

| メモリオーダ | 動作 |
| -- | -- |
| std::memory_order_relaxed | メモリオーダに対して何も作用しない。アトミックな操作のみを提供 |
| std::memory_order_release | Release fenceを実現する |  
| std::memory_order_acquire | Acquire fenceを実現する |  
| std::memory_order_acq_rel | Relase fence、Acquire fenceの両方を実現する |  
| std::memory_order_consume | Acquire fence(読み込んだ変数に依存する操作に対してのみ保証) |
| std::memory_order_seq_cst | 逐次一貫性を実現する |

std::memory_order_seq_cstは、逐次一貫性を実現することができる。 逐次一貫性を実現するということは、これが適用された時点でメモリ操作が全てのスレッドからプログラム順と同じ順で観測できることを示す。
atomicライブラリを理解するための基礎を記述できた。ここからatomicライブラリの説明に入る。

##### atomicライブラリ
atomic型は、組み込み型をテンプレート引数として受け取る。特殊化済みのtypedefも提供されている。同じであるためここでは、atomic<T>で表現する。
atomic型は、load(読み込み), store(書き込み)などの操作を提供しており、各操作に対して、メモリオーダを指定できるようになっている。
つまり、std::memory_order_acquireを指定することで、Acquire fenceを含めて提供することができる。指定しない場合はデフォルト動作として、std::memory_order_seq_cstが指定されたものとして動作する。
なお、atomic型に定義されている全てのオペレーションはフリー関数としても提供されている。例えば、atomic<T>.load()については、atomic_load()関数というように定義されている。
また、純粋にfence機能のみを提供する、atomic_thread_fnce関数もある。

2つのatomic操作(A,B)が、Release fence(A)、Acquire fence(B)を提供する場合で、Aの後に書き込んだ値をBで読み込めることを確認することでAまで実行されたことを保証できる。
よって、A -> B の関係が生まれる。
この関係を "Synchronizes with"関係と呼び、同期しているという。また、"Synchronizes with"である時、これを "happens before"関係とも呼ぶことができる。"A happens before B"というとき、Aは、Bが開始する前に完了していることを示す。

以下に、フェンスを使用した場合における"synchronizes with"を表現したサンプルを示す。

```c++
#include <atomic>
#include <iostream>

int A = 0;
int v1 = 0;
std::atomic<int> a(0);

void thread1() {
    A = 1;                                                  // a1
    std::atomic_thread_fence(std::memory_order_release);    // a2
    a.store(1, std::memory_order_relaxed);                  // a3
}

void thread2() {
    while( a.load(std::memory_order_relaxed) != 1);         // b1
    std::atomic_thread_fence(std::memory_order_acquire);    // b2
    v1 = A;                                                 // b3
}

int main() {
    std::thread t1(thread1);
    std::thread t2(thread2);

    t1.join();
    t2.join();
    std::cout << "v1 : " << v1 << std::endl;

    return 0;
}
```
上記の動作を考えて見る。まず、Release fenceにより a1 -> a2 であることは保証される。続いて、b1の終了条件はa3を実行していることであるため、a2 -> b2である。いて、Acquire fenceにより、b2 -> b3であることは保証される。これらより、a1 -> b3であることが導かれる。
ここで、a2 -> b2(a2 synchronizes with b2)の順序関係がない場合、a1, b3の命令は同時に実行され得る。つまりデータ競合が発生する可能性がある。
このように、フェンスを適切に用いることでロックフリーのプログラムが作成可能となる。しかし、全ての箇所で適切に同期されているプログラムを記述するのは非常に気をつけて記述する必要が有る。そのため、速度を気にしない場合はデフォルト動作である逐次一貫性(std::memory_order_seq_cst)を利用する方が良い。

フェンスを使用せず、atomic変数でフェンスを扱う場合のサンプルを以下に示す。

```c++
#include <atomic>
#include <iostream>

int A = 0;
int v1 = 0;
std::atomic<int> a(0);

void thread1() {
    A = 1;                                                  // a1
    a.store(1, std::memory_order_release);                  // a2
}

void thread2() {
    while( a.load(std::memory_order_acquire) != 1);         // b1
    v1 = A;                                                 // b2
}

int main() {
    std::thread t1(thread1);
    std::thread t2(thread2);

    t1.join();
    t2.join();
    std::cout << "v1 : " << v1 << std::endl;

    return 0;
}
```
上記についてもフェンス版と同様に考えることで、a1 -> b2 の順序関係により、 v1 == 1であることが保証される。


##### 参考文献
 1. Sorin, Daniel, Mark Hill, and David Wood. A Primer on Memory Consistency and Cache Coherence. Morgan & Claypool Publishers, 2011.
