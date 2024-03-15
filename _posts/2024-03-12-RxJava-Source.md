---
title: "RxJava Source Code Explained - Cold Observable"
date: 2024-03-01
---

## Introducation
Operators on a RxJava *Observable* can be composed declaratively following [Fluent Interface](https://en.wikipedia.org/wiki/Fluent_interface), so that method calls can be chained. For example,

```Java
Observable source = ***;
Observable op1 = source.operator1();
Observable op2 = op1.operator2();
Observable op3 = op2.operator3();
op3.subscribe(consumer);

// The above is equivalent to
source.operator1().operator2().operator3().subscribe(consumer);
```
For each operator/node in the chain, its left neighbor is the **upstream**/**source**, while the right is the **downstream**/**observer**. In the above example, 

| RxJava Nodes | Category |
| --- | --- |
| source | *Observable* of its *downstream*
| op1, op2, op3 | *Observer* of its *upstream* + *Observable* of its *downstream*
| consumer | *Observer* of its *upstream* |

## Basic Interfaces
*ObservableSource* is an abstraction of asynchronous sequences of multile items. *Observer*s can subscribe it to consume the items.
```Java
// Observale is consumable via an observer
public interface ObservableSource<T> {
    void subscribe(Observer<? super T> observer);
}
```
*Observable* can be converted using different operators, like map, filter, reduce. 

Also, we can define the threading model via *scheduleOn* and *observeOn*. Scheduler constrols where (which thread) the data emission or data observation will occur.
```Java
public abstract class Observable<T> implements ObservableSource<T> {
    // Map a Obserable to another of different types of items
    <R> Observable<R> map(Function<? super T, ? extends R> mapper);

    // Specify where Obervable itself will run to emit data
    Observable<T> subscrbeOn(Scheduler scheduler);

    // Specify where observers will observe the data
    Observable<T> observerOn(Scheduler scheduler);
}
```
*Observable* can *push* different types of notification towards its *Observer*s.
```Java
public interface Observer<T> {
    // Dispoasble allows anytime unsubscription
    void onSubscrbe(Disposable d);

    // Push the Observer with a new item to observe from Observable
    void onNext(T);

    // Used when Observable has encountered an error
    void onError(Throwable e);

    // Push the Observer with the signal that Obervable has finished
    void OnComplete();
}
```

**Note**: Disposable interface are ignored below for simplicity. Also, we will only focus on OnNext Observer method.

## Source Code Explained
Let us create a simplest Observable, which emits a single item upon subscription. It will notify/push its observer during emission.
```Java
public class Observable<T> {
    private final T value;

    public ObservableSingle(final T value) {
        this.value = value;
    }

    public void subscribe(Observer<? super T> observer) {
        observer.onSubscribe();
        observer.onNext(value);
        observer.onComplete();
    }

    public <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
        return new ObservableMap<>(this, mapper);
    }
}
```

### Map Operator
```Java
public class ObservableMap<T, R> implements Observable<T> {
    private Observable<T> source;
    private Function<? super T, ? extends R> mapper;

    public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends R> mapper) {
        this.source = source;
        this.mapper = mapper;
    }

    @Override
    public subscribe(Observer<? super T> observer) {
        ObserverMap<T, R> observerMap = new ObserverMap<T, R>(observer, mapper);
        source.subscribe(observerMap);
    }
}

public class ObserverMap<T, R> implements Observer<R> {
    private Function<? super T, ? extends R> mapper;
    private Observer<? super R> observer;

    public ObservableMap(Observer<? super R> observer, Function<? super T, ? extends R> mapper) {
        this.observer = observer;
        this.mapper = mapper;
    }

    public void OnNext(T value) {
        R r = mapper.apply(value);
        observer.onNext(r);
    }
}
```

## SubscribeOn
```Java
public class ObservableSubscribeOn<T> implements Observable<T> {
    private Scheduler scheduler;
    private Observable<T> source;

    public ObservableMap(ObservableSource<? super T> source, Scheduler scheduler) {
        this.source = source;
        this.scheduler = scheduler;
    }

    @Override
    public subscribe(Observer<? super T> observer) {
        scheduler.run([this, observer](){
            this.source.subscribe(observer);
        });
    }
}

```

## ObserveOn
```Java
// Same as ObservableMap

public class ObserverObserveOn<T> implements Observer<T> {
    private Scheduler scheduler;
    private Observer<T> observer;

    public ObservableMap(Observer<? super T> observer, Scheduler scheduler) {
        this.observer = observer;
        this.scheduler = scheduler;
    }

    // With ObserveOn, we run OnNext method in a given Scheduler instead of main thread
    public void OnNext(T value) {
        scheduler.run([this, value](){
            this.observer.onNext(value);
        });
    }
}
```
## Disposable

# Reference
* [手写极简版的RxJava](https://juejin.cn/post/6844903837849878536)
* [RxJava原理解析](https://juejin.cn/post/7042962823747469325)