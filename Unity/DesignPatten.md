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
 public string ProductName { get => productName; set => productName
 = value ; }
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