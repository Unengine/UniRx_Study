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

        await AsyncFunc();
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

    async UniTask<string> AsyncFunc()
    {
        //코루틴과 기존 Task도 await 가능하다.
        await SomeCoroutine();
        return await Task.Run(() => "AsyncFunc done");
    }

    IEnumerator SomeCoroutine()
    {
        yield return new WaitForSeconds(1);
        Debug.Log("Coroutine!");
    }
}
```

C#의 `Task`와 용법 차이가 거의 없다. 또한 `Task.AsUniTask` 확장 메서드로 기존의 `Task`를 `UniTask`로 만들 수 있다.

---

## `UniTask<T>`

`AsyncOperation`, `ResourceRequest`, `UnityWebRequestAsyncOperation`, `IEnumerator` 등 유니티에서 사용하는 여러 비동기 오브젝트를 `await` 할 수 있다.

또한, `UniTask.Delay`, `UniTask.Yield`, `UniTask.TimeOut`는 프레임 기준의 타이머이다.

## `Progress<T>`

유니티에서 사용하기 위한 경량화된 IProgress의 팩토리이다.

`Progress.Create<T>(Action<T> handler)` 메서드는 진행 상황의 변화마다 주어진 핸들러를 호출하는 `Progress` 객체를 생성한다.

## `ConfigureAwait(IProgress<float> progress)`

유니티의 AsyncOperation의 확장 메서드이다. Progress 업데이트 콜백을 받아들인다.

### 용법 (비동기 씬 로딩)

```cs
public async UniTask LoadScene()
{
    await SceneManager.LoadSceneAsync("AnotherScene").
    ConfigureAwait(
        Progress.Create<float>(x => Debug.Log(x))
        );
}
```

기존의 AsyncOperation 사용법보다 훨씬 구문이 간단하다. `ConfigureAwait`으로 현재 씬 로딩 작업의 진행도를 로그에 출력한다.

---

## 