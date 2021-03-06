### *thread*
---
#### 概要
C++11でスレッド関連のライブラリが追加された。追加された機能は、thread, mutex, スレッド同期(condition_variable), future/promiseパターンである。
ここでは、基本となるthreadクラスについて記述する。
C++11のthreadクラスはシンプルである。コンストラクタに別スレッドで処理させたい関数ポインタ、もしくは関数オブジェクトを指定するだけでスレッドが起動する。
引数を指定したい場合は、関数指定の後にそのまま記述すれば良い。
なお、threadクラスは実行した関数の戻り値は全て無視するため、実行結果を必要とする場合は別の方法で取得する必要が有る。

起動したスレッドは、必ずjoin()させなければならない。
規格策定中に暗黙的にjoin()する案も提案されていたが、意図しないバグを生み出す可能性があるとしてjoin()を必須としている。

本節で解説する機能を使用するには、ヘッダファイル **thread**をインクルードする。

以下に使用例を示す。

```c++
#include <thread>

void f() {}

struct func {
    void operator() (int a, int b) {}
};

int main() {
    std::thread t1(f);              // 関数ポインタを指定
    std::thread t2(func(), 1, 2);   // 関数オブジェクトを指定
    std::thread t3([](double a){
                    a = 1.0f;
                   },
                   0.0f);           // 関数オブジェクト(lambda関数)を指定
    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```

全てのスレッドにはユニークなが割り振られる。スレッドを起動した側では、threadクラスのget_id()を呼び出すことで取得できる。
スレッド内では、ネームスペースstd::this_threadに定義されたget_id()を呼び出すことで取得できる。
std::this_threadには他にも便利な関数が定義されている。以下に関数一覧を示す。

| 定義 | 効果 |
| -- | -- |
| get_id() | スレッド識別子を取得 |
| yield | コンテキストスイッチを発生させるヒントを提供する。 実装依存の動作。 |
| sleep_until(std::chrono::time_point) | 指定した日時までスリープ |
| sleep_for(std::chrono::duration)| 指定した時間スリープ |
※ std::this_thread::は省略

以下に使用例を示す。

```c++
#include <thread>
#include <chrono>

std::atomic<bool> flag(false);
void f() {
    using namespace std::chrono;
    std::this_thread::sleep_for(seconds(5));   //OK 5秒間sleep

    while(flag == false) {
        std::this_thread::yield;
    }

    std::cout << "f() end." << std::endl;
}

void g() {
    system_clock::time_point tp = system_clock::now();
    tp += minutes(7);
    std::this_thread::sleep_until(tp);         //OK 1分後までsleep

    flag = true;

    std::cout << "g() end." << std::endl;
}

int main() {
    std::thread t1(f);
    std::thread t2(g);

    t1.join();
    t2.join();
    return 0;
}
```
