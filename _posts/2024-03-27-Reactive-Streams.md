---
title: "Reactive Streams"
date: 2024-03-27
---

## Introducation
Java 9 (released in 2017) introduced the **Flow API** to support *Reactive Programming* natively. The feature composes four interfaces:

```Java
public final class Flow {

    /**
     * 
     */
    public static interface Publisher<T> {
        public void subscribe(Subscriber<? super T> subscriber);
    }

    /**
     * onSubscribe, onNext, onError and onComplete signaled to a Subscriber MUST be signaled serially.
     */
    public static interface Subscriber<T> {
        public void onSubscribe(Subscription subscription);

        public void onNext(T item);

        public void onError(Throwable throwable);

        public void onComplete();
    }

    public static interface Subscription {
        public void request(long n);

        public void cancel();
    }

    public static interface Processor<T,R> extends Subscriber<T>, Publisher<R> {
    }
}
```


Reactive Streams: Specification 

A fast data source (publisher) does not overwhelm the stream destination (subscriber)

backpressure

| Component | Specification |
| --- | --- |
| Publisher | |
| Subscriber | |
| Subscription | |
| Processor | |

These bounds must be respected by a publisher independent of whether the source it represents can be backpressured or not. In the case of sources whose production rate cannot be influenced—for example clock ticks or mouse movement—the publisher must choose to either buffer or drop elements to obey the imposed bounds.

## Reference
[Reactive Streams Home page](https://www.reactive-streams.org/)
[Reactive Streams Specification](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.4/README.md)
