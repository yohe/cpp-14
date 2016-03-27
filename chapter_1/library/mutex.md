### *mutex*
---
#### 概要
コア機能のマルチスレッドサポートにて、同じメモリに対して2つ以上の操作が発生し、どちらか一方が書き込み操作である場合にデータ競合となることを記載した。
プログラムが複数スレッドで動作する場合にはデータ競合が発生する可能性があるため、データ競合を回避する仕組みが必要となる。
本ライブラリは、データ競合に関する機能を提供する。
本ライブラリでは、Mutex, Locksオブジェクト, Call onceメカニズムを提供する。

本節で解説する機能を使用するには、ヘッダファイル **mutex**をインクルードする。

##### Mutex
Mutexは、特定の区間をアトミックに操作することを保証する機能を提供する。アトミック操作の開始と終了をメンバ関数lock()/unlock()で行う。
lock()の呼び出しで、呼び出し対象のMutexの所有権を得る。Mutexの所有権は１スレッドのみ得ることが可能である。unlock()により所有権を放棄する。
既に別スレッドが所有しているMutexに対してlock()を呼び出す場合、所有権が放棄された後に所有権を得ることになる。
lock()は所有権を得るまで待機することになるため、所有権が得られる場合のみ取得するtry_lock()もある。

Mutexには、再帰可能(自身で再度lock()可能)、タイムアウト有りの種類がある。以下に種類を示す。

| 定義 | 動作 |
| -- | -- |
| std::mutex |
| std::recursive_mutex |
| std::timed_mutex |
| std::recursive_timed_mutex |

以下に使用例を示す。

```c++
#include <thread>
#include <mutex>
#include <iostream>

std::mutex m;
std::timed_mutex tm;

void f() {
    bool own = m.try_lock();
    if( !own ) {
        std::cout << "[" << std::this_thread::get_id() << "] could not to lock." << std::endl;
        return;
    }
    std::cout << "[" << std::this_thread::get_id() << "] locked" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    m.unlock();
}

void ff() {
    // 待ち時間指定
    bool own = tm.try_lock_for(std::chrono::milliseconds(100));
    if( !own ) {
        std::cout << "[" << std::this_thread::get_id() << "] could not to lock." << std::endl;
        return;
    }
    std::cout << "[" << std::this_thread::get_id() << "] locked" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    m.unlock();
}

void fff() {
    // 日時指定
    auto t = std::chrono::system_clock::now() + std::chrono::seconds(5);
    bool own = tm.try_lock_until(std::chrono::milliseconds(100));
    if( !own ) {
        std::cout << "[" << std::this_thread::get_id() << "] could not to lock." << std::endl;
        return;
    }
    std::cout << "[" << std::this_thread::get_id() << "] locked" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    m.unlock();
}

int main() {
    std::thread t1(f);
    // 出力が被らないように100ms待つ
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    std::thread t2(f);
    t1.join();
    t2.join();

    std::thread t3(ff);
    std::thread t4(ff);
    t3.join();
    t4.join();
```

##### Locksオブジェクト
Mutexの提供するlock()/unlock()による排他制御はunlock()漏れによるデッドロックを引き起こす可能性がある。そのため、mutexをそのままで使用することは推奨されない。
そこで、RAIIイディオムを適用したLocksオブジェクトを使用する。
Lockオブジェクトはコンストラクタ時にlock()を呼び出し、デストラクタでunlock()を呼び出すためunlock()漏れが起きない。

Locksオブジェクトには、lock_guardクラス, unique_lockクラスの２種類がある。lock_guardは、lock/unlockのみのシンプルな機能のみを提供し、
unique_lockは、try_lockにも対応した高機能なクラスである。

以下に使用例を示す。

```c++
#include <thread>
#include <mutex>
#include <iostream>

std::mutex m;
std::timed_mutex tm;

void f() {
    std::lock_guard<std::mutex> lock(m);
    std::cout << "locked" << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
}

void ff() {
    std::unique_lock<std::timed_mutex> lock(tm, std::chrono::seconds(1));
    std::cout << "locked" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
}

int main() {
    std::thread t1(f);
    std::thread t2(f);
    t1.join();
    t2.join();
    std::thread t3(ff);
    std::thread t4(ff);
    t3.join();
    t4.join();
```

例では示していないが、unique_lockの場合、コンストラクタではlock()せずに、明示的にlock()することも可能である。
この場合でもデストラクタでunlock()されるため安全である。


##### Call once
Call onceとは、一度だけ実行されるような処理を実現する機能である。例えば、Singletonパターンで一度だけ初期化するような場合に使用することもできる。
この機能は、once_flagオブジェクトと実行する関数ポインタ、もしくは関数オブジェクトをcall_once関数に渡して利用する。

以下に使用例を示す.

```c++
#include <thread>
#include <mutex>

std::once_flag once;

void init(int a, int& b) { }

void f() {
    int a, b;
    std::call_once(once, init, a, std::ref(b));
}
```

呼び出す関数に引数がある場合は、呼び出す関数の後に列挙する。
