# RxDart

[![Build Status](https://travis-ci.org/ReactiveX/rxdart.svg?branch=master)](https://travis-ci.org/ReactiveX/rxdart)
[![codecov](https://codecov.io/gh/ReactiveX/rxdart/branch/master/graph/badge.svg)](https://codecov.io/gh/ReactiveX/rxdart)
[![Pub](https://img.shields.io/pub/v/rxdart.svg)](https://pub.dartlang.org/packages/rxdart)
[![Gitter](https://img.shields.io/gitter/room/ReactiveX/rxdart.svg)](https://gitter.im/ReactiveX/rxdart)

## About

RxDartは、[ReactiveX](http://reactivex.io/)をベースにしたGoogle Dart言語用のリアクティブ関数プログラミングライブラリです。
Google Dartにはすぐれた[Stream](https://api.dartlang.org/dev/2.0.0-dev.58.0/dart-async/Stream-class.html) API が付属していますが、RxDartはこのAPIに代わるものを提供しようとするのではなく、RxDartはその上に機能を追加します。

## Version

Dart 1.0は、v0.15.x以前でサポートされています。v0.16.x以降にはもはや後方互換性が失われているので、Dart SDK 2.0が必須です。

## RxDartの使い方

### 例: コナミコマンドの読み込み

```dart
void main() {
  const konamiKeyCodes = const <int>[
    KeyCode.UP,
    KeyCode.UP,
    KeyCode.DOWN,
    KeyCode.DOWN,
    KeyCode.LEFT,
    KeyCode.RIGHT,
    KeyCode.LEFT,
    KeyCode.RIGHT,
    KeyCode.B,
    KeyCode.A];

  final result = querySelector('#result');
  final keyUp = new Observable<KeyboardEvent>(document.onKeyUp);

  keyUp
    .map((event) => event.keyCode)
    .bufferCount(10, 1)
    .where((lastTenKeyCodes) => const IterableEquality<int>().equals(lastTenKeyCodes, konamiKeyCodes))
    .listen((_) => result.innerHtml = 'KONAMI!');
}
```

## APIの概要

### Objects

- [Observable](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable-class.html)
- [PublishSubject](https://www.dartdocs.org/documentation/rxdart/latest/rx/PublishSubject-class.html)
- [BehaviorSubject](https://www.dartdocs.org/documentation/rxdart/latest/rx/BehaviorSubject-class.html)
- [ReplaySubject](https://www.dartdocs.org/documentation/rxdart/latest/rx/ReplaySubject-class.html)

### Observable

RxDartのObservablesは、DartのStreamクラスを拡張しています。これは２つの大きな意味があります。
- すべての[`Stream`クラスに定義されたメソッド](https://api.dartlang.org/stable/1.21.1/dart-async/Stream-class.html#instance-methods)は、RxDartの`Observables`クラス上にもすべて実装されています。
- すべての`Observables`は、 `Stream`を入力として受け取るAPIにも指定することができます
- その他の重要な違いは、[Observableクラス](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable-class.html)の一部として文書化されています。

Finally, the Observable class & operators are simple wrappers around `Stream` and `StreamTransformer` classes. All underlying implementations can be used free of the Observable class, and are exposed in their own libraries. They are linked to below.

### Instantiation

Generally speaking, creating a new Observable is either done by wrapping a Dart Stream using the top-level constructor `new Observable()`, or by calling a factory method on the Observable class.
But to better support Dart's strong mode, `combineLatest` and `zip` have been pulled apart into fixed-length constructors. 
These methods are supplied as static methods, since Dart's factory methods don't support generic types.

###### Usage
```dart
var myObservable = new Observable(myStream);
```

#### `Observable`で利用可能なファクトリメソッド
- [concat](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.concat.html) / [ConcatStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/ConcatStream-class.html)
- [concatEager](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.concat.html) / [ConcatEagerStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/ConcatEagerStream-class.html)
- [defer](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.defer.html) / [DeferStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/DeferStream-class.html)
- [error](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.error.html) / [ErrorStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/ErrorStream-class.html)
- [just](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.just.html)
- [merge](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.merge.html) / [MergeStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/MergeStream-class.html)
- [never](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.never.html) / [NeverStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/NeverStream-class.html)
- [periodic](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.periodic.html)
- [race](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.race.html) / [RaceStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/RaceStream-class.html)
- [retry](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.retry.html) / [RetryStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/RetryStream-class.html)
- [timer](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/Observable.timer.html) / [TimerStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/TimerStream-class.html)

###### Usage
```dart
// 複数のStreamを1本に合成
var myObservable = new Observable.merge([myFirstStream, mySecondStream]);
```

##### `Observable`で利用可能な静的メソッド

- [combineLatest](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/combineLatest2.html) / [CombineLatestStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/CombineLatestStream-class.html) (combineLatest2, combineLatest... combineLatest9) <br />最後の要素同士を合成する
- [range](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/range.html) / [RangeStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/RangeStream-class.html)
- [tween](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/tween.html) / [TweenStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/TweenStream-class.html)
- [zip](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/zip2.html) / [ZipStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_streams/ZipStream-class.html) (zip2, zip3, zip4, ..., zip9)<br />要素をペアに合成する

###### Usage
```dart
var myObservable = Observable.combineLatest3(
    myFirstStream, 
    mySecondStream, 
    myThirdStream, 
    (firstData, secondData, thirdData) => print("$firstData $secondData $thirdData"));
```

### Transformations
    
##### 利用可能なメソッド
- [buffer](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/buffer.html) / [BufferStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html)
- [bufferCount](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/bufferCount.html) / [BufferStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onCount](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onCount.html)
- [bufferFuture](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/bufferFuture.html) / [BufferStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onFuture](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onFuture.html)
- [bufferTest](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/bufferTest.html) / [BufferStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onTest](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onTest.html)
- [bufferTime](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/bufferTime.html) / [BufferStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onTime](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onTime.html)
- [bufferWhen](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/bufferWhen.html) / / [BufferStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onStream.html)
- [concatMap](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/concatMap.html) (alias for `asyncExpand`)
- [concatWith](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/concatWith.html)
- [debounce](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/debounce.html) / [DebounceStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DebounceStreamTransformer-class.html)
- [delay](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/delay.html) / [DelayStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DelayStreamTransformer-class.html)
- [dematerialize](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/dematerialize.html) / [DematerializeStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DematerializeStreamTransformer-class.html)
- [distinctUnique](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/distinctUnique.html) / [DistinctUniqueStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DistinctUniqueStreamTransformer-class.html)
- [doOnCancel](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnCancel.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [doOnData](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnData.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [doOnDone](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnDone.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [doOnEach](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnEach.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [doOnError](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnError.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [doOnListen](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnListen.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [doOnPause](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnPause.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [doOnResume](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/doOnResume.html) / [DoStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/DoStreamTransformer-class.html)
- [exhaustMap](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/exhaustMap.html) / [ExhaustMapStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/ExhaustMapStreamTransformer-class.html)
- [flatMap](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/flatMap.html) / [FlatMapStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/FlatMapStreamTransformer-class.html)
- [interval](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/interval.html) / [IntervalStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/IntervalStreamTransformer-class.html)
- [materialize](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/materialize.html) / [MaterializeStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/MaterializeStreamTransformer-class.html)
- [mergeWith](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/mergeWith.html)
- [max](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/max.html) / [StreamMaxFuture](https://www.dartdocs.org/documentation/rxdart/latest/rx_futures/StreamMaxFuture-class.html)
- [min](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/min.html) / [StreamMinFuture](https://www.dartdocs.org/documentation/rxdart/latest/rx_futures/StreamMinFuture-class.html)
- [repeat](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/repeat.html) / [RepeatStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/RepeatStreamTransformer-class.html)
- [sample](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/sample.html) / [SampleStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/SampleStreamTransformer-class.html)
- [scan](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/scan.html) / [ScanStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/ScanStreamTransformer-class.html)
- [skipUntil](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/skipUntil.html) / [SkipUntilStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/SkipUntilStreamTransformer-class.html)
- [startWith](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/startWith.html) / [StartWithStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/StartWithStreamTransformer-class.html)
- [startWithMany](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/startWithMany.html) / [StartWithManyStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/StartWithManyStreamTransformer-class.html) 
- [switchMap](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/switchMap.html) / [SwitchMapStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/SwitchMapStreamTransformer-class.html)
- [takeUntil](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/takeUntil.html) / [TakeUntilStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/TakeUntilStreamTransformer-class.html)
- [timeInterval](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/timeInterval.html) / [TimeIntervalStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/TimeIntervalStreamTransformer-class.html)
- [timestamp](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/timestamp.html) / [TimestampStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/TimestampStreamTransformer-class.html)
- [throttle](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/throttle.html) / [ThrottleStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/ThrottleStreamTransformer-class.html)
- [window](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/window.html) / [WindowStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/WindowStreamTransformer-class.html)
- [windowCount](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/windowCount.html) / [WindowStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onCount](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onCount.html)
- [windowFuture](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/windowFuture.html) / [WindowStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onFuture](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onFuture.html)
- [windowTest](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/windowTest.html) / [WindowStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onTest](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onTest.html)
- [windowTime](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/windowTime.html) / [WindowStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onTime](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onTime.html)
- [windowWhen](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/windowWhen.html) / [WindowStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/BufferStreamTransformer-class.html) / [onStream](https://www.dartdocs.org/documentation/rxdart/latest/rx_samplers/onStream.html)
- [withLatestFrom](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/withLatestFrom.html) / [WithLatestFromStreamTransformer](https://www.dartdocs.org/documentation/rxdart/latest/rx_transformers/WithLatestFromStreamTransformer-class.html)
- [zipWith](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable/zipWith.html)

Dart言語のStream APIから提供されるメソッド(`first`, `asyncMap`など)の全容は、[DartDocs](https://www.dartdocs.org/documentation/rxdart/latest/rx/Observable-class.html#instance-methods)を参照してください

###### Usage

```dart
var myObservable = new Observable(myStream)
    .bufferCount(5)
    .distinct();
```

## 例

`example`フォルダにコマンドラインからも実行可能なサンプルが用意されています。

### ブラウザでの実行例
 
In order to run the web examples, please follow these steps:

  1. Clone this repo and enter the directory
  2. Run `pub get`
  3. Run `pub serve example/web`
  4. Navigate to [http://localhost:8080](http://localhost:8080) in your browser

### コマンドラインでの実行例

In order to run the command line example, please follow these steps:

  1. Clone this repo and enter the directory
  2. Run `pub get`
  3. Run `dart example/bin/fibonacci.dart 10`
  
### Flutterでの例
  
#### Install Flutter

In order to run the flutter example, you must have Flutter installed. For installation instructions, view the online
[documentation](http://flutter.io/).

#### Run the app

  1. Open up an Android Emulator, the iOS Simulator, or connect an appropriate mobile device for debugging.
  2. Open up a terminal
  3. `cd` into the `example/flutter/github_search` directory
  4. Run `flutter doctor` to ensure you have all Flutter dependencies working.
  5. Run `flutter packages get`
  6. Run `flutter run`

## Notable References
- [Documentation on the Dart Stream class](https://api.dartlang.org/stable/1.21.1/dart-async/Stream-class.html)
- [Tutorial on working with Streams in Dart](https://www.dartlang.org/tutorials/language/streams)
- [ReactiveX (Rx)](http://reactivex.io/)

## Changelog

Refer to the [Changelog](https://github.com/frankpepermans/rxdart/blob/master/CHANGELOG.md) to get all release notes.
