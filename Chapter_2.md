# UniRx Study

## Custom Observable 만들기

직접 Observable한 스트림을 반환하는 메서드를 작성해보자.

```cs

protected void Start()
{
	MyInterval(TimeSpan.FromSeconds(1), 5)
		.SubscribeOn(Scheduler.ThreadPool)
		.ObserveOnMainThread()
		.Subscribe(elapsed => Debug.Log(elapsed))
		.AddTo(this);
}

protected IObservable<double> MyInterval(TimeSpan period, float duration)
{
	return Observable.Create<double>(observer =>
	{
		double elapsed = 0;
		while (elapsed < duration)
		{
			elapsed += period.TotalSeconds;
			observer.OnNext(elapsed);
			Thread.Sleep((int)period.TotalMilliseconds);
		}
		observer.OnCompleted();
		return Disposable.Empty;
	});
}
```


이 코드는 간단한 타이머를 구현한 코드이다. 게임 시작 시 5초동안 1초마다 타이머가 진행된 시간을 로그에 출력한다. 스트림은 1초(주기)마다 타이머가 진행된 시간을 옵저버에게 전달한다.

이 스트림을 Start 함수에서 구독하는데, 이 때 스트림은 쓰레드풀에서 구해온 다른 쓰레드에서 작동하며, 감시는 메인 쓰레드에서 진행한다.

메시지가 도착할 때마다 메인 쓰레드의 옵저버는 타이머가 진행된 시간을 얻어와 로그를 출력한다.

그러나 이 방법은 위험하다. 옵저버와 스트림의 쓰레드가 서로 다르기 때문에 동작이 꼬일 수 있으며, 게임이 일시정지 되어도 계속 스트림의 쓰레드가 동작한다.

---

안전한 방법이 하나 있다. 유니티 내에서 원시적인 비동기식 프로그래밍을 지원해주는 코루틴 또한 UniRx에서 감시할 수 있다!

그말인 즉슨 직접 코루틴을 작성해 Observe 할 수 있다는 것이다.

```cs
protected void Start()
{
	Observable.FromCoroutine<double>(
		observer => CMyInterval(observer, TimeSpan.FromSeconds(0.5), 5))
		.Subscribe(elapsed => Debug.Log(elapsed))
		.AddTo(this);
}

protected IEnumerator CMyInterval(IObserver<double> observer, TimeSpan period, float duration)
{
	double elapsed = 0;
	while (elapsed < duration)
	{
		elapsed += period.TotalSeconds;
		observer.OnNext(elapsed);

		yield return Observable.ToYieldInstruction(
			Observable.Timer(TimeSpan.FromSeconds(period.TotalSeconds))
		);
	}
	observer.OnCompleted();
}
```

이 코드는 게임 시작 시 5초동안 0.5초마다 타이머가 진행된 시간을 로그에 출력한다.

코루틴으로 동작하고 있기 때문에 게임이 일시정지 될 시 타이머가 멈춘다.

유의할 점은 옵저버에게 데이터를 전달하기 위해 인자에 옵저버를 꼭 포함시켜야 한다는 것이다.

여기서 옵저버에게 데이터를 전달하는 부분은 `observer.OnNext(elapsed)` 이다.


```cs
yield return Observable.ToYieldInstruction(
		Observable.Timer(TimeSpan.FromSeconds(period.TotalSeconds)
	);
```
덤으로 이 부분에선 스트림을 yieldable한 Enumerator로 바꿔준다.

---

## 여러 오퍼레이터

스트림(`IObservable<T>`)은 여러 오퍼레이터로 필터링될 수 있다.

오늘은 자주 쓰이는 것들을 공부했다.

`Where<T>(Func<T, bool> predicate)` 인자로 받아들여진 조건식에 따라 메시지를 필터링한다.

`Select<TR>(Func<T, TR> selector)` 인자로 받아들여진 함수로 메시지를 가공할 수 있게 한다.

`First<T>()` 첫 메시지만 발행하고 끝낸다(Complete).

`Last<T>()` 스트림의 마지막 메시지만 발행하고 끝낸다.

`Take(int count)` 인자로 주어진 수 만큼 메시지를 발행한다.

`TakeWhile(Func<T, bool> predicate)` 인자로 준 조건식이 불만족할 때까지 메시지를 발행한다.

`TakeUntil(IObservable<T> other)` 인자로 준 스트림에서 메시지가 발행될 때까지 메시지를 발행한다.

`Skip(int count)` 인자로 주어진 수 만큼 메시지 발행을 무시한다.

`SkipWhile(Func<T, bool> predicate)` 인자로 준 조건식이 불만족할 때까지 메시지를 무시한다.

`SkipUntil(IObservable<T> other)` 인자로 준 스트림에서 메시지가 발행될 때까지 메시지를 무시한다.

`DistinctUntilChanged()` 메시지의 값이 변경될 때만 메시지를 발행한다.