# UniRx Study

## 오퍼레이터 - 좀 더 어려운 것

스트림의 메세지를 필터링 할 오퍼레이터가 정말 많은데 하나같이 중요한 것 같다.

```cs
var doubleClickStream = this.UpdateAsObservable()
	.Where(_ => Input.GetMouseButtonDown(0));

doubleClickStream.Buffer(doubleClickStream.Throttle(TimeSpan.FromMilliseconds(500)))
	.Where(clkCount => clkCount.Count >= 2)
	.Subscribe(_ => Debug.Log("Double Clicked"));
```

`Throttle(TimeSpan dueTime)` 메시지가 집중되었을 시 마지막 메시지만 발행한다. **첫 메시지부터 인자로 준 시간까지**를 집중된 시점으로 본다.

`Buffer(int Count)` 인자로 준 횟수만큼 메시지를 모았다 횟수가 충족되면 메시지를 발행한다.

`Buffer(IObservable<Unit> windowBoundaries)` 인자로 스트림에서 메시지 발생 시 메시지를 발행한다.

이 예제에서는, 마지막으로 클릭한 지 0.5초 (500 밀리초) 후에 Throttle의 집중 시점이 끝나므로 한 번이라도 클릭할 시 0.5초 후 메시지가 발행된다.

이 때 Buffer의 인자로 준 스트림에서 메시지가 발행됐으므로 Buffer했던 스트림에서 메시지가 발행된다.

그동안 마우스 클릭 이벤트가 버퍼에 저장되고, 버퍼에 저장된 이벤트의 리스트의 Count를 검사함으로써 0.5초 내에 마우스가 두 번 이상 클릭됬는 지 필터링할 수 있는 것이다.