### *future*
---
#### 概要
C++11で追加されたスレッドに関連した機能であり、future/promiseパターンを提供する。C++でのfuture/promiseパターンは、ある関数で実行された処理結果(値 or exception)を同じスレッド、もしくは別のスレッドで実行時とは異なるタイミングで回収することを可能にする。基本的にマルチスレッドで使用するがシングルスレッドでも利用可能である。
また、future/promiseパターンを簡単に利用するためのラッパ機能(async, package_task)も提供している。

本節で解説する機能を使用するには、ヘッダファイル **future**をインクルードする。

以下に、それぞれの機能について説明する。

##### future/promise

futureは、処理結果を回収する機能を提供する。もし処理結果が準備できていない場合は準備できるまで待機する。
準備ができるまで永遠に待ち続けるため、タイムアウトありの待機も可能である。タイムアウトは、日時指定と期間指定が可能である。
promiseはfutureの処理結果を準備する役割を持ち、このためpromiseとfutureは互いに紐付いており、futureオブジェクトはpromiseオブジェクトから取得する。
なおfuture、およびpromiseはコピー不可でありムーブのみ可能である。
ムーブ、もしくは1度でも値を回収したfutureの状態は不正であり、不正状態でのメンバ関数呼び出しは未定義である。
shared_futureクラスの場合、値の回収後も不正状態にはならない。

以下に使用例を示す。

```c++
#include <thread>
#include <future>
void promise(std::promise<int> p) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    p.set_value(0);         // 処理結果を登録
    p.set_exception(std::runtime_error("error"));         // 処理結果を登録
}

int main() {
    std::promise<int> p;
    std::future<int> future = p.get_future();

    std::thread t1(promise, std::move(p));
    // 準備ができるのを2秒待つ
    std::future_status st = future.wait_for(std::chrono::seconds(2));
    if ( st != std::future_status::timeout ) {
        int ret = future.get();
        future.get();                           //NG 不正状態でのget()
    }
    t1.join();

    return 0;
}
```

処理結果を複数のスレッドで参照した場合もある。この場合は、shared_futureを使用する。
shared_futureは、futureから変換して取得する。変換後のfutureは不正状態である。
shared_futureは、複数回のget() が可能である事、コピーが許可される事以外はfutureと同じである。

```c++
#include <thread>
#include <future>
#include <iostream>

void promise(std::promise<int> p) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    p.set_value(1);
}

void get(std::shared_future<int> sf) {
    std::cout << sf.get() << std::endl;
}

int main() {
    std::promise<int> p;
    std::future<int> future = p.get_future();
    std::shared_future<int> sf = future.share();

    std::thread t1(promise, std::move(p));
    std::thread t2(get, sf);
    std::thread t3(get, sf);

    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```

##### async
asyncは、promiseを作成してthreadを起動し、関数の戻り値をpromiseに設定するまでを代行するラッパ関数である。また、threadを起動するかを選択する起動オプションを指定する事も可能性である。
以下に標準で規定されている起動オプションを示す。

| 定義 | 効果 |
| -- | -- |
| launch::deffered | 指定した関数の実行を遅延させる。futureにアクセス(get or wait)するまで関数は実行されない |
| launch::async | 新しくスレッドを作成し、指定した関数を非同期に実行する |
なお、両方を指定した場合(launch::deffered | launch::async)、どちらを選択するかは実装依存である。
起動オプションを指定しない場合は、両方を指定した場合と等しい。

以下に使用例を示す。

```c++
#include <future>

int async(int num) {
    std::cout << "async=" << std::this_thread::get_id() << std::endl;
    return num;
}

int main() {
    std::cout << "main=" << std::this_thread::get_id() << std::endl;
    std::future<int> f = std::async(async, 100);
    std::cout << f.get() << std::endl;

    f = std::async(std::launch::deferred, async, 1000);
    std::cout << f.get() << std::endl;
    return 0;
}
```

出力結果を以下に示す。

```
main=0x7fff7c172300
async=0x10289f000
100
async=0x7fff7c172300
1000
```

##### packaged_task
packaged_taskは、promiseを高級にしたようなものである。
packaged_taskは実行可能なオブジェクトであり、構築時に実行したい関数を指定しておく。そしてpackaged_taskを実行する時に指定した関数の引数を渡す。
実行した関数の戻り値は自動的にpromiseに設定される。
packaged_taskに指定可能な関数は、通常の関数、および関数オブジェクト(function, bind, lambda関数, etc...)である。

以下に使用例を示す。

```c++
#include <future>
#include <thread>
#include <iostream>

int async(int num) {
    std::cout << "async=" << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(num));
    return num;
}

int main() {
    std::packaged_task<int(int)> task(async);
    std::future<int> f = task.get_future();
    task(10);
    std::cout << f.get() << std::endl;
    return 0;
}
```

