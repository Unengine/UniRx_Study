# UniRx Study
## UniRx.Async

UniRx에서 사용하는 비동기 처리는 `UniRx.Async`라는 클래스에서 취급한다. 이는 C#의 `Task`를 유니티에서 사용하기 좋도록 최적화 한 것이다. C#의 `Task`는 매우 무겁고, `SynchronizationContext`에 의존해 제약이 많다. 그러나 UniRx에서의 `Task`인 `UniTask`는 매우 가볍고, `SynchronizationContext` 의존도도 없다.

---

## 예제 코드

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UniRx;
using UniRx.Async;
using System.Threading.Tasks;

public class AsyncEx : MonoBehaviour
{
    // Start is called before the first frame update
    async void Start()
    {
		var lightWorkTask = LightWork();
		var heavyWorkTask = HeavyWork();

		var allTasks = Task.WhenAll(lightWorkTask, heavyWorkTask);
		var strs = await allTasks;
		foreach (var s in strs)
			Debug.Log(s);

		var uniTaskA = UniTaskA();
		var uniTaskB = UniTaskB();
		var uniTasks = new List<UniTask<string>>
		{
			uniTaskA,
			uniTaskB
		};

		var allUniTasks = UniTask.WhenAll(uniTasks);
		strs = await allUniTasks;
		foreach (var s in strs)
			Debug.Log(s);

	
	}
	
    async Task<string> LightWork()
	{
		return await Task.Run(() => "Job's done!");
	}

	async Task<string> HeavyWork()
	{
		await Task.Delay(2000);
		return await Task.Run(() => "Heavy job's done!");
	}

	async UniTask<string> UniTaskA()
	{
		return await UniTask.Run(() => "UniTask job A!");
	}

	async UniTask<string> UniTaskB()
	{
		await UniTask.Delay(3000);
		return await UniTask.Run(() => "UniTask job B!");
	}
}
```

