# UniRx TIL

## Caution!
불친절할 수 있음.

뇌피셜의 내포가 있을 수 있음. (필자 나름의 이해가 곁들여짐)

내용이 정확하지 않을 수 있음!

살짝 이해될락말락 한 상태에서 작성했기 때문에 설명을 대충 함. (깊이가 얕음)

---

## UniRx란?
C#에서 반응형 프로그래밍(Reactive Programming)을 위한 확장인 Rx.Net을 유니티에서 사용할 수 있도록 재구현한 것. Rx.Net은 유니티에서 잘 작동하지 않기 때문에 (퍼포먼스, IL2CPP 빌드 문제 등) UniRx에서는 Rx.Net의 유니티 내 사용 상 문제점을 고치고 유니티를 위한 기능을 추가하였다.

---

## 반응형 프로그래밍?

간단하게 설명하자면 데이터를 스트림(Stream)에서 감시(Observe)함으로써 값의 변화를 체크하고, 이에 따른 동작을 수행할 수 있도록 하는 프로그래밍 패러다임이다.

ex)

```cs
public Image panel;

protected void Start()
{
	var uiClickStream = panel.OnPointerClickAsObservable()
		.Subscribe(_ => Debug.Log("UI Clicked!"))
}
```

UI 패널을 클릭했을 때를 감시하고, (OnPointerClickAsObservable)

클릭되었을 시 이벤트가 발생하여 로그를 출력한다.

---

## 설치 방법

간단하다. 에셋 스토어에 UniRx를 검색하고 다운받아 임포트 하면 끝이다. 스크립트 뿐이라 용량도 얼마 안 된다.

---

## 쓰는 이유?

반응형 프로그래밍은 용이하다. 비동기 처리는 말할 것도 없고, 특정한 이벤트(주로 입력)를 기다리거나, **값의 변화를 용이하게 감시**할 수 있다. 또한, **선언형 프로그래밍**을 지향하기 때문에 게임 내의 시간에 따른 변화를 감지해내고 특정 동작을 수행하는 데 큰 편리함을 주기도 한다. 또한, Start, Awake같은 함수 내에서 지속적인 동작을 '선언'해둘 수 있기 때문에 Update 함수를 깨끗하게 유지할 수 있고, LINQ문을 잘 지원하기 때문에 구문이 매우 간단해진다.

---
## 선행 지식 - Observer Pattern

객체의 상태 변화를 관찰하는 관찰자들 (Observers)의 목록을 객체에 등록해 상태 변화가 있을 때 마다 메서드 등을 이용해 객체가 직접 각 관찰자들에게 통보하는 패턴.

### 대강의 이벤트, 델리게이트 명칭? (UniRx에서 사용하는 옵저버 패턴)

Subscribe - 관찰자 등록

OnNext - 상태 변화를 알림

OnCompleted - 모든 데이터가 발송된 것을 알림. 단 한번만 발생하고, 더 이상 OnNext가 발생하지 않는다.

OnError - 상태 변화 때 오류가 발생했을 시 알림. OnNext, OnCompleted가 실행되지 않고 Observable의 실행 종료.

Dispose - 더 이상 데이터가 발생하지 않도록 구독을 해지하는 함수. 보통 OnCompleted 이후 발생한다.

UnSubscribe - 관찰자 등록 해제. C#의 옵저버 패턴 인터페이스에서 보였다. UniRx에서는 그냥 Dispose 쓰는듯?

---

## Observable (관찰 가능한 객체)을 생성하는 방법

여러 가지가 있기 때문에 높은 빈도로 사용 되거나 대표성을 띄는 극히 일부만 서술한다.

Component.UpdateAsObservable - 업데이트 시마다 이벤트 발생

UnityEngine.EventSystems.UIBehaviour.OnPointerClickAsObservable - UI 요소 클릭할 때 마다 발생.

Observable.Interval(TimeSpan period) - 주어진 TimeSpan을 주기로 이벤트 발생

---

## 코드 구경하면서 본 여러 (확장)메서드들의 정체

일단 Observable들은 LINQ문으로 필터링이 가능하다는 것을 알아두자.

이런식으로.

```cs
var clickStream = this.UpdateAsObservable()
	.Where(_ => Input.GetMouseButtonDown(0) && Filter(Input.mousePosition))
	.Select(pos => Input.mousePosition).
	Subscribe(pos => Debug.Log(pos));

	...

protected bool Filter(Vector3 pos) => pos.x >= 100 && pos.x <= 1180 && pos.y >= 100 && pos.y <= 620;
```

마우스 버튼이 눌러졌을 시 마우스 위치를 필터링한 후, 조건에 맞을 시 마우스 위치를 출력하는 예시이다.

---

```cs
//(대기 가능, 확장)
System.IObservable<T>.Subscribe<T>(System.Action<T> onNext)
```
스트림에서 이벤트를 받은 후 할 행동을 정의한다.

실제로 이벤트를 받았을 때 인자로 받은 Action을 잘 실행한다.

---

```cs
//(확장)
System.IDisposable.AddTo<System.IDisposable>(Component gameObjectComponent)
```
Observable이 파괴될 때 인자로 준 컴포넌트의 게임오브젝트가 파괴됐을 때 자동으로 Dispose를 해준다. 그러나 OnCompleted는 호출되지 않는다.

예전엔 Observable.EveryUpdate 라고 있었는데 MainThreadDispatcher에서 **코루틴 실행 타이밍**에 이벤트를 뿌려주기 때문에 Dispose를 직접 해줬어야 했다.

(지금은 이미 그게 Deprecated다. IObservable<T>.UpdateAsObservable이란 비슷한 게 있는데 그건 Dispose를 자동으로 해준다.)

아무튼 이러한 케이스를 보면 분명 몇몇 Observable들은 게임오브젝트가 파괴됐을 때 Dispose를 자동으로 호출시켜주지 않을 것이다. 또한 직접 Observable을 만들었지만 Dispose가 자동으로 호출되지 않을 수 있다. 그렇기 때문에 이 메서드가 존재하는 것이다.

---

비슷한 용도의 메서드가 하나 더 있다.

```cs
//(대기 가능, 확장) <- 주목! SubScribe 전에 해야함.
System.IObservable<T>.TakeUntilDestroy<T>(Component target)
```

약간의 차이점이라면 AddTo는 인자로 준 **컴포넌트가 부착된 게임 오브젝트**가 파괴됐을 때 Dispose 하는 것이고, TakeUntilDestroy는 인자로 준 **컴포넌트** 자체가 파괴되었을 때 Dispose 하는 것이다. 또한, OnCompleted가 호출된다.

---