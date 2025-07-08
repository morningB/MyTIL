### Kiss(Keep it simple, stupid) 원칙

- 단순하게 생각하기
- 디자인 패턴 적용은 더 많은 비용을 들게 함(**신중하게 설계**)

### SOILD

- **단일 책임**(Single responsibility)
- **개방-폐쇄**(Open-closed)
- **리스코프 치환**(Liskov substitution)
- **인터페이스 분리**(Interface segregation)
- **종속성 역전**(Dependency inversion)

### 단일 책임 원칙

- **클래스** 또는 **함수**는 **오직 한가지만 책임**져야함
- **특정 부분만 캡슐화**
- 하나의 큰 것 보단 **여러 개의 작은 클래스를 만드**는 편이 좋다는 것
- 비추천하는 코드
    
    ```csharp
    public class UnrefactoredPlayer : MonoBehaviour
    {
     [SerializeField] private string inputAxisName;
     [SerializeField] private float positionMultiplier;
     private float yPosition;
     private AudioSource bounceSfx;
     private void Start()
     {
     bounceSfx = GetComponent<AudioSource>();
     }
     private void Update()
     {
     float delta = Input.GetAxis(inputAxisName) * Time.deltaTime;
     yPosition = Mathf.Clamp(yPosition + delta, -1, 1);
     transform.position = new Vector3(transform.position.x, yPosition * 
     positionMultiplier, transform.position.z);
     }
     private void OnTriggerEnter(Collider other)
     {
     bounceSfx.Play();
     }
    } 
    
    ```
    
    - 움직임, 오디오, 이펙트가 다 하나의 포함
    - 하나의 클래스의 너무 많은 기능이 포함
- 추천 코드
    
    ```csharp
    [RequireComponent(typeof(PlayerAudio), typeof(PlayerInput), 
    typeof(PlayerMovement))]
    public class Player : MonoBehaviour
    {
     [SerializeField] private PlayerAudio playerAudio;
     [SerializeField] private PlayerInput playerInput;
     [SerializeField] private PlayerMovement playerMovement;
     private void Start()
     {
     playerAudio = GetComponent<PlayerAudio>();
     playerInput = GetComponent<PlayerInput>();
     playerMovement = GetComponent<PlayerMovement>();
     }
    }
    public class PlayerAudio : MonoBehaviour
    {
    	// 오디오 부분
    }
    public class PlayerInput : MonoBehaviour
    {
    	// 입력처리 부분
    }
    public class PlayerMovement : MonoBehaviour
    {
    	// 움직임 부분
    } 
    ```
    
- 단일 책임 원칙을 목표로 할 경우
1. **가독성**: 짧은(200~300줄) 클래스를 작성하라
2. **확장성**: 작은 클래스가 상속하기 더 쉬움.
3. **재사용성**: 다른 부분에서 사용할 수 있게 작은 모듈로 디자인

### 개방-폐쇄 원칙

- 확장에는 개방, 수정에는 폐쇄
- 대표적인 예: 셰이프 영역을 계산하는 클래스
- 원본을 수정하지 않고 기능 추가하도록 구조화
- 추천하지 않는 코드
    
    ```csharp
    public class AreaCalculator 
    {
     public float GetRectangleArea(Rectangle rectangle)
     {
     return rectangle.width * rectangle.height;
     }
     public float GetCircleArea(Circle circle)
     {
     return circle.radius * circle.radius * Mathf.PI;
     }
    }
    public class Rectangle
    {
     public float width;
     public float height;
    }
    public class Circle
    {
     public float radius;
    } 
    ```
    
    - 기능을 충분히 수행하지만, **더 많은 셰이프를 추가한다면 각각의 메서드를 생성해야됨.**
    - **20각형**까지 있다면 매우 귀찮고 각 셰이프를 처리하도록 **if문도 여러개**가 필요함
        
    <img width="1398" height="1017" alt="Image" src="https://github.com/user-attachments/assets/90de9080-a58a-4567-9d31-3fcdbfc4bba8" />
        
- 해결하기 위한 `abstract Shape` 클래스 정의
    
    ```csharp
    public abstract class Shape
    {
    	public abstract float CalculateArea();
    }
    ```
    
- Rectangle과 Circle이 **상속 받음**
    - override해서 사용
    
    ```csharp
    public class Rectangle : Shape
    {
     public float width;
     public float height;
     public override float CalculateArea()
     {
     return width * height;
     }
    }
    public class Circle : Shape
    {
     public float radius;
     public override float CalculateArea()
     {
     return radius * radius * Mathf.PI; 
     }
    }
    ```
    
- **각 셰이프를 사용할 클래스**
    
    ```csharp
    public class AreaCalculator 
    {
     public float GetArea(Shape shape)
    	 {
    		 return shape.CalculateArea();
    	 {
     }
    public class AreaCalculator : MonoBehaviour
    { 
      private void Start()
     {
     Debug.Log(GetArea(new RectAngle { width = 2, height = 3 }));
     Debug.Log(GetArea(new Circle { radius = 3 }));
     }
     public float GetArea(Shape shape)
     {
     return shape.CalculateArea();
     }
    }
    ```
    
    - `{ width = 2, height = 3 }`
    - **생성자 없이 객체 초기화** 구문으로 값 할당 가능
        - 필드가 많아도 필요한 것만 할 수 있음
        - 선택적 선택 가능
- 새로운 셰잎이 필요하면 상속받아 정의하면 됨
- 디버깅도 쉽고 오류가 발생해도 AreaCalculator를 검토 안해도 됨
- 기존 코드의 변화가 없기에 **새로운 코드만 확인**하면 됨

### 리스코프 치환 원칙

- 정의: **서브타입**(자식 클래스)이 언제나 **슈퍼타입**(기본 클래스)으로 교체 가능해야 함
    - 갑자기 자식 클래스를 없애고 부모를 사용해도 오류가 없어야됨
- Vehicle은 트럭과 차의 기본형

    <img width="1398" height="1035" alt="Image" src="https://github.com/user-attachments/assets/51824d67-6614-47ad-814b-9aed9c8b330f" />

```csharp
public class Vehicle
{
 public float speed = 100;
 public Vector3 direction;
 public void GoForward()
 {
 ...
 }
 public void Reverse()
 {
 ...
 }
 public void TurnRight()
 {
 ...
 }
 public void TurnLeft()
 {
 ...
 }
}
```

- 경로에 따라 움직이도록 구현 가능
    
    ```csharp
    public class Navigator
    {
     public void Move(Vehicle vehicle)
     {
    	 vehicle.GoForward();
    	 vehicle.TurnLeft();
    	 vehicle.GoForward();
    	 vehicle.TurnRight();
    	 vehicle.GoForward();
     }
    }
    ```
    
- But 기차는 `TurnLeft`와 `Right`이 필요 없음
    
    <img width="1398" height="1066" alt="Image" src="https://github.com/user-attachments/assets/b9a2a5f0-53c6-4606-bf63-086f8af3f837" />
    
    - Navigator의 Move를 기차에 전달해도 불필요한 구현이 생김
    - 이렇게 **특정 유형**을 **하위 유형과 교체할 수 없**다면 리스코프 치환 **원칙을 위반**하게 됨

### 리스코프를 **준수하기 위한 팁**

1. 서브 클래스를 구현할 때 기능을 제거하면 위반 가능성 높아짐
    - `NotImplementedException`은 원칙을 위반했다는  의미이며, 메서드를 비워 두는 경우도 마찬가지
    - 자식이 기본처럼 작동하지 않는다면 명시적으로 오류가 없어도 위반한 것
2. **추상화**를 **단순하게 유지**해야됨
    - 기본 클래스가 **최대한 단순**해야됨
    - 일반적인 기능만 표현
3. 자식은 부모와 **동일한 공용 멤버를 가져야**함(같은 동작 의미)
    - 공용 멤버는 호출 시 동일한 서명을 갖고 동일한 동작을 취해야 함
4. 클래스 계층 구조를 수립할 때 **클래스 API를 고려**해야됨
    - 모두 차량이어도 Car와 Train은 서로 다른 부모를 상속하는게 더 나을수도
    - **실질적인 분류가 항상 클래스 계층 구조와 일치하진 않음**
5. 상속보다는 합성을 우선시
    - 상속을 통해 기능전달하기 보다는 인터페이스나 별도 클래스 제작
    - **상속:** is-a, **구현:** has-a
    
    <img width="1398" height="824" alt="Image" src="https://github.com/user-attachments/assets/1f89293d-903a-4ed9-a4d4-a1e636ed7cb0" />
    
- Vehicle 유형을 삭제하고 인터페이스로 옮겨야됨
    
    ```csharp
    public interface ITurnable 
    {
     public void TurnRight();
     public void TurnLeft();
    }
    public interface IMovable
    {
     public void GoForward();
     public void Reverse();
    }
    ```
    
    - **단일 책임 원칙**과도 맞닿음
- Road와 Rail을 만들면 더 철저히 지킴

<img width="1398" height="1025" alt="Image" src="https://github.com/user-attachments/assets/053204cd-f8ea-4631-ad7d-cb50f18169ac" />

```csharp
public class RoadVehicle : IMovable, ITurnable
{
 public float speed = 100f;
 public float turnSpeed = 5f;
 public virtual void GoForward()
 {
 ...
 }
 public virtual void Reverse()
 {
 ... 
 }
 public virtual void TurnLeft()
 {
 ... 
 }
 public virtual void TurnRight()
 {
 ... 
 }
}
public class RailVehicle : IMovable
{
 public float speed = 100;
 public virtual void GoForward()
 {
 ...
 }
 public virtual void Reverse()
 {
 ...
 }
}
public class Car : RoadVehicle
{
 ...
}
public class Train : RailVehicle
{
 ...
}
```

- 기능이 인터페이스를 통해 실행
- 추가 내용
    - 이러한 사고방식은 직관적이지 않은 것처럼 보일 수 있는데, 사람들이 실제 세상에 대해 가지는 확고한 가정이
    있기 때문입니다.
    - 소프트웨어 개발에서는 이를 원-타원 문제(Circle–ellipse problem)라고 합니다.
    - 모든 실제 등가 관계가 상속으로 전환되지는 않습니다.
    - 소프트웨어 디자인으로 진행하려는 것은 실제 세상에 대한 사전 지식이 아닌, 클래스 계층 구조라는 사실을 기억하세요

### 인터페이스 분리 원칙

- 정의: 어떠한 클라이언트도 자신이 사용하지 않는 메서드에 강제로 종속될 수 없음
- 즉, 인터페이스의 규모가 커지지 않게 해야됨
    - 코드 길이를 짧게 유지하라는 맥락
- 다양한 플레이어 유닛이 있는 전략 게임
    - 각 유닛에는 체력, 속도 등 스탯 존재
- 모든 유닛이 아래 코드의 기능을 구현하는 인터페이스 제작한다면?
    
    ```csharp
    
     public float Health { get; set; }
     public int Defense { get; set; }
     public void Die();
     public void TakeDamage();
     public void RestoreHealth();
     public float MoveSpeed { get; set; }
     public float Acceleration { get; set; }
     public void GoForward();
     public void Reverse();
     public void TurnLeft();
     public void TurnRight();
     public int Strength { get; set; }
     public int Dexterity { get; set; }
     public int Endurance { get; set; }
    } 
    ```
    
- 부술 수 있는 상자나 통은 체력은 필요하지만 Move는 필요없음
- 작은 단위의 인터페이스를 여러개 만듦

<img width="1398" height="952" alt="Image" src="https://github.com/user-attachments/assets/6515b36b-b5cc-452f-ac7f-07690a9d4493" />

```csharp
public interface IMovable
{
 public float MoveSpeed { get; set; }
 public float Acceleration { get; set; }
 public void GoForward();
 public void Reverse();
 public void TurnLeft();
 public void TurnRight();
}
public interface IDamageable 
{
 public float Health { get; set; }
 public int Defense { get; set; }
 public void Die();
 public void TakeDamage();
 public void RestoreHealth();
}
public interface IUnitStats 
{
 public int Strength { get; set; }
 public int Dexterity { get; set; }
 public int Endurance { get; set; }
} 
public interface IExplodable 
{
 public float Mass { get; set; }
 public float ExplosiveForce { get; set; }
 public float FuseDelay { get; set; }
 public void Explode();
} 
```

### 인터페이스 직렬화SerializeField

- 인터페이스에 `SerializeField` 나 public을 해도 인스펙터에 표기 안됨
- MonoBehaviour 또는 스크립터블 오브젝트를 상속받는 클래스만 작동하도록 설계
- 인터페이스는 본질적으로 추상적이며 데이터가 없지만 직렬화 사용 가능
- 인터페이스를 구현하는 구체적인 오브젝트(MonoBehaviour,스크립터블 오브젝트 등)의 레퍼런스를 직렬화
- 런타임에 is 키워드를 사용하여 직렬화된 오브젝트를 확인하고 캐스트