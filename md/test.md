# Hello, RxCpp

saichi suzuki

---

## What Rx

![Rx](img/rx.jpg)

>>>

## Word1

* Observable
  * ストリーム
* Message
  * ストリームから飛んでくる値のこと
* Event
  * メッセージが飛んでくるonNext/onComplete/onError
* Subscribe
  * 購読

>>>

## Word2

* Subject
  * 通知ストリーム
    * subscriber: 発信者
    * observable: 受信者
      * subscriberはon_nextで発行し, on_completedで終了する
      * HotなObservable
* Observe
  * 監視
    * ObserveEveryValueChanged

---

## 例1(タイマー)

```cpp
#include "rxcpp/rx.hpp"
#include <chrono>
```

```cpp
auto period = std::chrono::milliseconds(1000);
auto scheduler = rxcpp::observe_on_new_thread();

auto intervalValues = rxcpp::observable<>::interval(period, scheduler).
intervalValues.subscribe([](long s) {
    printf("%li: 秒経過", s);
}, [](){printf("Interval OnCompleted\n");});

intervalValues.unsubscribe(); // 監視終了
```

>>>

## 例2(値監視)

```cpp
auto period = std::chrono::milliseconds(0);
auto scheduler = rxcpp::observe_on_new_thread();

auto bgmIsPlayingObserver = rxcpp::observable<>::interval(period, scheduler).
map([=](long s) { // フレーム数をisBGMに変換
    return SoundManager::getInstance()->isBackgroundMusic();
}).
distinct_until_changed(); // Messageに変更があれば
bgmIsPlayingObserver.subscribe([](bool f) { // 購読
    printf("Change OnNext: %i", f);
}, [](){printf("Change OnCompleted"); });
```

>>>

## 例3(時間変更時、Viewに通知する)

```cpp
// 毎フレーム: Hotなストリーム用意
_timeUpdateObservable = rxcpp::observable<>::interval(std::chrono::milliseconds(0), rxcpp::observe_on_new_thread()).
publish();


// 毎分: 時計更新
_minuteObservable = _timeUpdateObservable.
map([=](long s) {
    return _currentTime->tm_min;
}).
distinct_until_changed().
skip(1).
subscribe([=](long s) {
    _currentTimeSubject.get_subscriber().on_next(_currentTime); //時間データをSubjectに送信
}, [](){printf("Interval OnCompleted\n");
});
```

---

## Reference

[Github](https://github.com/Reactive-Extensions/RxCpp)

[Github/doc](https://github.com/Reactive-Extensions/RxCpp/tree/master/Rx/v2/examples/doxygen)

[cocos導入法参考](http://qiita.com/gamako/items/861631e6c74ff3e19f33)

[Rxcpp memo(qiita)](http://qiita.com/masuhajime/items/2fd1d96493d1151dde02)

[逆引き](http://wilfrem.github.io/learn_rx/operators.html)

[RxSwift](https://speakerdeck.com/ra1028/reactive-thinking-in-swift)

