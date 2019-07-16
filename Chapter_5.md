# UniRx Study
## ReactiveProperty와 Subject


ReactiveProperty는 변수의 값이 변경되었을 시 메시지를 발행하는 스트림이다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UniRx;

public class ReactivePropEx : MonoBehaviour
{
	public ReactiveProperty<int> data;
	public ReactiveProperty<TestDataClass> testData;
	public UnityEngine.UI.Button btn;

    // Start is called before the first frame update
    void Start()
    {
		data = new ReactiveProperty<int>(0);
		btn = GetComponent<UnityEngine.UI.Button>();
		btn.onClick.AddListener(() => ++data.Value);

		data.Subscribe(data => Debug.Log(data));
	}
}
```

이 예제 코드는 씬 안의 버튼이 클릭되었을 시 버튼의 리스너는 `data.Value`의 값을 변동시키고, ReactiveProperty가 변동을 감지해 메시지를 발행해 로그에 데이터를 출력한다.


단, 유의할 점은 클래스나 구조체의 내부 값이 변경되었다고 메시지를 발행하지 않는다.

다시 말해, 원시 자료형(int, double ...)등의 경우 값 자체의 변경은 메시지를 발행하지만, 클래스가 구조체의 내부 필드의 변동은 메시지를 발행하지 않는다는 것이다.

```cs
public class TestDataClass
{
	public int data;
	public TestDataClass(int data)
	{
		this.data = data;
	}
}
.
.
.
public class ReactivePropEx : MonoBehaviour
{
	public ReactiveProperty<TestDataClass> data;
	public UnityEngine.UI.Button btn;

    // Start is called before the first frame update
    void Start()
    {
		dta = new ReactiveProperty<TestDataClass>(new TestDataClass(0));

		btn = GetComponent<UnityEngine.UI.Button>();
		btn.onClick.AddListener(() =>
		{
			++testData.Value.data;
			//testData.Value = new TestDataClass(testData.Value.data + 1);
		});
		data.Subscribe(data => Debug.Log(data.data));
	}
}
```

해당 코드는 필드 초기화시에만 로그를 단 한 번만 출력하고 버튼 클릭 시에도 로그가 출력되지 않는다. 그러나 AddListener 부분의 람다식의 첫 번째 줄을 주석 처리하고, 두 번째 줄의 주석 처리를 없애주면 버튼 클릭시 마다 로그가 출력된다.

즉, 메시지는 오직 ReactiveProperty.Value의 변경에만 발행된다. 클래스나 구조체를 ReactiveProperty로 사용하는 것은 개인적으로 좋지 않은 용법이라 생각하지만, 만일 사용하게 된다면 이 점을 유의해야 한다.

---

