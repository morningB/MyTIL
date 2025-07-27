## 디자인 패턴

### 팩토리 패턴

<img width="1304" height="1043" alt="Image" src="https://github.com/user-attachments/assets/29ca4615-ed3d-4537-905b-dc313b2e10cf" />

하나 이상의 제품을 생산할 수 있음

- 공장이라는 특수 오브젝트 지정
- 하나의 레벨에서 생산과 관련된 많은 세부사항을 캡슐화 → 수정 용이
- 각 제품의 고유 로직은 공장에서 보이지 않게 해도 됨 → 확장성 향상

```csharp
public interface IProduct
{
	public string ProductName { get; set; }
	public void Initialize();
}
public abstract class Factory : MonoBehaviour
{
	public abstract IProduct GetProduct(Vector3 position);
	// 모든 공장이 공유하는 메서드
	…
} 
```

- 제품은 각 메서드에 맞는 특정 템플릿을 따라야 하지만,
- 그 외에는 어떤 기능도 공유할 필요가 없습니다.
- 따라서 IProduct 인터페이스를 정의합니다.
- 구조

<img width="1403" height="696" alt="Image" src="https://github.com/user-attachments/assets/d0a96e97-711a-4093-8cec-8c11bcfcc2a9" />

- IProduct만 준수하면 원하는만큼 정의 가능
- Factory에는 IProduct를 반환하는 메서드 존재
- 서브클래스로 파생시키면 다른 제품 획득
- 특정 위치에서 더 쉽게 인스턴스화 가능
- 

```csharp
public class ProductA : MonoBehaviour, IProduct
{
	[SerializeField] private string productName = “ProductA”;
	public string ProductName { get => productName; set => productName = value ; }
	private ParticleSystem particleSystem;
	public void Initialize()
	{
		// 이 제품에 대한 모든 고유 로직
		gameObject.name = productName;
		particleSystem = GetComponentInChildren<ParticleSystem>();
		particleSystem?.Stop();
		particleSystem?.Play();
	}
}
public class ConcreteFactoryA : Factory
{
	[SerializeField] private ProductA productPrefab;
	public override IProduct GetProduct(Vector3 position)
	{
	// 프리팹 인스턴스를 생성하고 제품 컴포넌트 가져오기
		GameObject instance = Instantiate(productPrefab.gameObject,
		position, Quaternion.identity);
		ProductA newProduct = instance.GetComponent<ProductA>();
		// 각 제품에 자체 로직 포함
		newProduct.Initialize();
		return newProduct;
	}
}
```

- ProductA는 ConcreteFactoryA가 복사본을 인스턴스화할 때 재생

### 장점

- 많은 제품을 설계할 때 유용
- 제품을 새로 정의해도 기존 유형 수정 XX

### 단점

- 패턴을 구현하기 위해 다수의 클래스와 서브 클래스를 만들어야함
- 약간의 오버헤드 발생

### 개선 방안

1. 딕셔너리를 사용하여 제품 검색: 키-값 페어로 딕셔너리에 저장
    - 제품 또는 제품 공장을 더 편리하게 검색 가능
2. 공장을 정적 클래스로 설정: 사용은 더 쉽지만 추가 설정 필요
    - 정적 클래스는 Inspector에 표시되지 않으므로 제품의 컬렉션도 정적으로 만들어야함
3.  GameObject와 MonoBehaviour가 아닌 요소에 적용
    - 프리팹 또는 컴포넌트로 한정하지 말기
    - C#이면 모든지 가능
4. 오브젝트 풀 패턴 결합
    - 반드시 새로운 오브젝트를 만드는 건 아님
    - 기존의 오브젝트도 검색할 수 있음
    - 많은 오브젝트를 한번에 인스턴스(총알)는 오브젝트 풀 활용

### 오브젝트 풀

- 최적의 성능을 위해 게임 씬 내 수많은 오브젝트 라이프사이클을 관리해야됨.
- C#의 자동 메모리 관리 시스템은 가비지 컬렉터를 제공
    - BUT 자주 생성되거나 파괴되면 렉 걸릴 수 있음
- 생성, 파괴 대신 미리 초기화하고 비활성화된 오브젝트의 푸ㄹ을 유지하는 것
    - 인스턴스화 필요 없음
- ParticleSystem이 오브젝트 풀과 유사
- 오브젝트 풀의 크기를 설명하거나 프리팹에 대한 설명, 풀 자체를 형성할 컬렉션을 설정

```csharp
public class ObjectPool : MonoBehaviour
{
	 [SerializeField] private int initPoolSize;
	 [SerializeField] private PooledObject objectToPool;
	 // 풀링된 오브젝트를 컬렉션에 저장
	 private Stack<PooledObject> stack;
	 private void Start()
	 {
		 SetupPool();
	 }
	 // 풀 생성(지연을 인지할 수 없을 때 호출)
	 private void SetupPool()
	 {
		 stack = new Stack<PooledObject>();
		 PooledObject instance = null;
		 for (int i = 0; i < initPoolSize; i++)
		 {
			 instance = Instantiate(objectToPool);
			 instance.Pool = this;
			 instance.gameObject.SetActive(false);
			 stack.Push(instance);
		 }
	 }
	  // 풀에서 첫 번째 액티브 게임 오브젝트를 반환합니다.
	 public PooledObject GetPooledObject()
	 {
		 // 풀이 충분히 크지 않으면 새로운 PooledObjects를 인스턴스화
		 if (stack.Count == 0)
		 {
			 PooledObject newInstance = Instantiate(object ToPool);
			 newInstance.Pool = this;
			 return newInstance;
		 }
	 // 그렇지 않으면 목록에서 다음 항목을 가져옵니다.
		 PooledObject nextInstance = stack.Pop();
		 nextInstance.gameObject.SetActive(true);
		 return nextInstance;
	 }
	 public void ReturnToPool(PooledObject PooledObject)
	 {
		 stack.Push(PooledObject);
		 PooledObject.gameObject.SetActive(false);
	 }
}
```

- `SetupPool` 로 오브젝트 풀 채우기
    - 인스턴스로 만들고, 이를 스택에 넣는다.
- `GetPooledObject` 이 비어있다면 새로운 풀을 생성
- `GetPooledObject` 를 호출하는 클라이언트는 풀링된 오브젝트를 회전/이동시켜야함

```csharp
public class PooledObject : MonoBehaviour
{
	private ObjectPool pool;
	public ObjectPool Pool { get => pool; set => pool = value; }
	public void Release()
	{
		pool.ReturnToPool(this);
	}
} 
```

- `ObjectPool`을 참조하기 위한 작은 `PooledObject`를 가짐
- `ReturnToPool` 을 호출하면 오브젝트가 비활성화 되고 풀 대기열로 반환

- 이것처럼 하면 사용자는 총알을 생성하는 대신 오브젝트 풀 메서드를 호출

<img width="1357" height="1038" alt="Image" src="https://github.com/user-attachments/assets/b03ec051-f9e7-40e6-9de5-e0357c59ae1a" />

 풀링된 오브젝트 비활성화 및 재사용

- 수백의 총알 발사 연출처럼 보임
- 실제로는 총알을 비활성화하고 재사용하는 것
- 풀의 크기는 활성화된 오브젝트를 동시에 표시할 수 있을만큼 커야함.

### UnityEngine.Pool

- Unity 내부에서 자체적으로 만듬

```csharp
public class RevisedGun : MonoBehaviour
{
	…
	// Unity 2021 이상 버전에서 사용 가능한 스택 기반 ObjectPool
	private IObjectPool<RevisedProjectile> ObjectPool;
	// 이미 풀에 있는 기존 항목을 반환하려 할 때 예외를 반환
	[SerializeField] private bool collectionCheck = true;
	// 풀의 용량과 최대 크기를 제어하는 추가 옵션
	[SerializeField] private int defaultCapacity = 20;
	[SerializeField] private int maxSize = 100;
	private void Awake()
	{
		ObjectPool = new ObjectPool<RevisedProjectile>
		(CreateProjectile,OnGetFromPool, OnReleaseToPool, OnDestroyPooledObject, collectionCheck, defaultCapacity, maxSize);
	}
	// 오브젝트 풀을 채울 항목을 만들 때 호출됨
	private RevisedProjectile CreateProjectile()
	{
		RevisedProjectile projectileInstance = Instantiate(projectilePrefab);
		projectileInstance.ObjectPool = ObjectPool;
		return projectileInstance;
	}
	// 오브젝트 풀로 항목을 반환할 때 호출됨
	private void OnReleaseToPool(RevisedProjectile PooledObject)
	{
		PooledObject.gameObject.SetActive(false);
	}
	// 오브젝트 풀에서 다음 항목을 검색할 때 호출됨
	private void OnGetFromPool(RevisedProjectile PooledObject)
	{
		PooledObject.gameObject.SetActive(true);
	}
	// 풀링된 항목의 최대 개수를 초과할 때 호출됨(풀링된 오브젝트 파괴)
	private void OnDestroyPooledObject(RevisedProjectile PooledObject)
	{
		Destroy(PooledObject.gameObject);
	}
	private void FixedUpdate()
	{
	…
	}
}
```

- 스크립트 내용의 대부분이 원본 ExampleGun 스크립트에서 작동합니다.
- 하지만 이제 다음 시점에 로직을 설정하는 유용한 기능이 ObjectPool 생성자에 포함됩니다.
- 풀을 채우기 위해 풀링된 항목을 먼저 생성하기
- 풀에서 항목 가져오기
- 풀로 항목 반환하기
- 풀링된 오브젝트 파괴하기(예: 최대 한도에 도달한 경우)

1. 생성자로 전달할 해당 메서드를 정의해야 합니다.
2. 빌트인 ObjectPool에서 어떤 식으로 기본 풀 크기 및 최대 풀 크기 옵션도 포함하는지 참고하세요. 
3. 최대 풀크기를 초과하는 항목은 자동 파괴 행동을 트리거하여 메모리 사용량을 억제합니다.
4. 발사체 스크립트는 ObjectPool에 레퍼런스를 유지하도록 일부 수정됩니다. 
5. 이렇게 하면 오브젝트를 풀로 더 간편하게 릴리스할 수 있습니다.

```csharp
public class RevisedProjectile : MonoBehaviour
{
	private IObjectPool<RevisedProjectile> ObjectPool;
	// 발사체에 ObjectPool에 대한 레퍼런스를 제공하는 공용 프로퍼티
	public IObjectPool<RevisedProjectile> ObjectPool { set => object Pool = value; }
} 
```

### 오브젝트 풀 장점
- 가비지 컬렉션 오버헤드 감소: 오브젝트를 생성 및 파괴하지 않고 재사용하면 가비지 컬렉션의 필요성이 감소
	- 따라서 런타임에 성능 스파이크나 끊김 현상을 방지

- 성능 향상: 오브젝트를 미리 초기화하고 필요할 때마다 다시 활성화하면 슈팅 게임처럼 페이스가 빠른 게임에서 성능을 향상

- 초기화 과정 최적화: 오브젝트 생성 과정을 덜 중요한 시점으로 분산하여 리소스 및 애플리케이션 시작 시간을 최적화

### 오브젝트 풀 단점

- 복잡도 증가: 오브젝트 풀에는 더 많은 관리가 필요. 
	- 오브젝트를 적절히 초기화하고 릴리스하지 않으면 오류나 버그가 발생

- 메모리 사용량: 오브젝트 풀을 사용하면 가비지 컬렉션이 줄어들지만 정적 메모리 사용량이 늘어
	- 오브젝트 풀은 오브젝트를 사용하지 않을 때도 사전 정의된 만큼의 오브젝트를 메모리에 저장 
	- 프로젝트 수요에 따라 적절하게 풀 크기를 조정해야됨
- 관리 필요성 증가: 최적의 오브젝트 풀 크기를 결정하는 일은 어려움

### 개선 방안
- 정적 또는 싱글톤으로 설정: 
	- 다양한 소스에서 풀링된 오브젝트를 생성해야 하는 경우에는 오브젝트 풀을 정적으로 재사용할 수 있도록 설정하기 
	- 그러면 애플리케이션의 어디에서든 액세스할 수 있으나, 인스펙터 사용은 불가능
	- 또는 오브젝트 풀 패턴을 싱글톤으로 설정하면 어디에서든 액세스하여 간편하게 사용

- 딕셔너리로 다수의 풀 관리: 
	- 많은 수의 다양한 프리팹을 풀링해야 하는 경우, 개별적인 풀에 저장하고 키-값 페어를 저장하면 쿼리할 풀을 알 수 있습니다
	- (프리팹의 InstanceID는 고유 키로 작동할 수 있음).

- 사용되지 않는 게임 오브젝트를 창의적으로 제거: 
	- 오브젝트 풀을 효과적으로 활용하는 방법 중 하나는 사용되지 않는 오브젝트를 숨기고 풀로 반환하는 것 	
	- 가능한 기회를 모두 활용해 풀링된 오브젝트(화면 바깥에 있거나 폭발에 의해 숨겨진 오브젝트 등)를 비활성화하기

- 오류 확인: 
	- 이미 풀에 있는 오브젝트를 릴리스하지 말기
	- ObjectPool<T>의 인스턴스를 생성할 때 collectionCheck 파라미터를 true로 설정 가능
	- 이렇게 하면 이미 풀에 있는 오브젝트를 반환하려 할 때 에디터에서 예외가 발생

- 최대 크기/제한 추가: 
	- 풀링된 오브젝트가 많으면 메모리를 소모.
	- ObjectPool 생성자의 maxSize 파라미터를 사용하여 풀 크기를 제한.

### 싱글톤 패턴

- GoF에 따르면 싱글톤 패턴은 다음을 수행
    - 클래스가 자체의 인스턴스 하나만을 인스턴스화하도록 보장
    - 해당하는 하나의 인스턴스에 대한 글로벌 액세스 제공
- 전체 씬에서 행동을 조정하는 오브젝트가 정확히 하나만 필요할 때 유용
- 예를 들면 씬에 메인 게임 루프를 총괄하는 게임 관리자가 딱 하나만 필요

- 만드는 방법

```csharp
public class SimpleSingleton : MonoBehaviour
{
	public static SimpleSingleton Instance;
	private void Awake()
	{
		if (Instance == null)
		{
			Instance = this; 
		}
		else
		{
			Destroy(gameObject); 
		}
	}
}
```
- 기존에 존재한다면 제거
- 싱글톤 문제점
	- 계층 구조에서 싱글톤을 설정해야됨
	- 보통 관리자 역할을 하므로 DontDestroyOnLoad를 사용하여 지속성을 가지게 함
- 지연 인스턴스를 통해 자동으로 생성
- 개선된 싱글톤 패턴
```csharp
public class Singleton : MonoBehaviour
{
	private static Singleton Instance;
	public static Singleton Instance
	{
		get
			{
				if (Instance == null)
				{
					SetupInstance();
				}
				return Instance;
			}
	}
	
	private void Awake()
	{
		if (Instance == null)
		{
			Instance = this;
			DontDestroyOnLoad(this.gameObject);
		}
		else
		{
			Destroy(gameObject);
		}
	}
	private static void SetupInstance()
	{
		Instance = FindObjectOfType<Singleton>();
		if (Instance == null)
		{
			GameObject gameObj = new GameObject();
			gameObj.name = “Singleton”;
			Instance = gameObj.AddComponent<Singleton>();
			DontDestroyOnLoad(gameObj);
		}
	}
}
```