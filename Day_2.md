# UniRx Study

## Custom Observable 만들기

유니티 내에서 가장 원시적인 비동기식 프로그래밍 비스무리한 것을 지원해주는 코루틴 또한 UniRx에서 감시할 수 있다!

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

이 코드는 간단한 타이머를 구현한 코드이다. 게임 시작 시 5초동안 0.5초마다 타이머가 진행된 시간을 로그에 출력한다.

유의할 점은 옵저버에게 데이터를 전달하기 위해 인자에 옵저버를 꼭 포함시켜야 한다는 것이다.

여기서 옵저버에게 데이터를 전달하는 부분은 `observer.OnNext(elapsed)` 이다.