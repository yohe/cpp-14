### *condition variable*
---
#### 概要
condition variableとは、wait()/notify()によるスレッド間同期機能を提供する。
condition variableには、std::unique_lockを受け取るcondition_variableと、BasicLockableを満たす任意のmutexを受け取るcondition_variable_anyがある。
機能的には変わらないので、基本的にcondition_variableを使用すれば良い。

condition_variableには、一般的なwait/notify機能が提供されている。wait機能には、通常のwait, 日時、期間指定のwaitが提供されている。
また、notify機能には、対象のcondition_variableでwaitしている内の1スレッドを起こすnotify_one(), 
対象のcondition_variableでwaitしている全てのスレッドを起こすnotify_all()が提供されている。
なお、wait状態のスレッドが存在しない時にnotifyを行う場合、notifyは無視される。そのため、wait前のnotifyが発生した場合、waitによるデッドロックを注意する必要がある。

本機能を使用する場合、ヘッダファイル**condition_variable**をインクルードすれば良い。

以下に使用例を記載する。

```c++
#include <thread>
#include <mutex>
#include <condition_variable>

struct X {
    void f() {
        std::cout << "ok" << std::endl;
    }
};

X* x = nullptr;

std::mutex m;
std::condition_variable cond;

void do_notify() {
    x = new X();
    cond.notify_one();
}

void do_sync_process() {
    std::unique_lock<std::mutex> lock(m);
    cond.wait(lock);

    x->f();
}

int main() {
    std::thread t1(do_sync_process);
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::thread t2(do_notify);


    t1.join();
    t2.join();

    return 0;
}
```
実は、上記プログラムはバグを含んでいる。バグは"Spurious wakeup"という事象によって引き起こされる。
Spurious wakeupとは、waitにより待機しているスレッドが、notifyを受け取る前に起きてしまう現象である。[[参考](https://en.wikipedia.org/wiki/Spurious_wakeup)]
バグを回避するためには、 Spurious wakeupが発生した場合に再度waitするように実装する、
もしくはcondition_variableが提供している 関数オブジェクトPredicate を加えたwaitを用いるようにする。
Predicateはbool値を返す関数オブジェクトで、falseを返す限り再waitが行われる。以下のような実装である。

```c++
while(!pred())
    wait(lock);
```

以下に関数オブジェクトを用いた使用例を示す。

```c++
#include <thread>
#include <mutex>
#include <condition_variable>

struct X {
    void f() {
        std::cout << "ok" << std::endl;
    }
};

X* x = nullptr;

bool ready=false;
std::mutex m;
std::condition_variable cond;

void do_notify() {
    x = new X();
    ready=true;
    cond.notify_one();
}

void do_sync_process() {
    std::unique_lock<std::mutex> lock(m);
    cond.wait(lock, []{ return ready; } );

    x->f();
}

int main() {
    std::thread t1(do_sync_process);
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::thread t2(do_notify);


    t1.join();
    t2.join();

    return 0;
}
```
