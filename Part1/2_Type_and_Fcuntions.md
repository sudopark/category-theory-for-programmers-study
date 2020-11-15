## 2. Types and Functions



### 2.1 Who Needs Types?

Static, Strong 한 타입이 Dynamic, Weak 한 타입보다 프로그램에서 더 유용하다

- 정적이고 강력한 타입이 원숭이가 무작위로 타자를 쳐서 세익스피어의 작품을 만들어내는것을 방지하는 스펠체커 역할을 한다.

  

### 2.2 Types Are About Composability

카테고리 이론은 화살표(사상)의 합성에 관한것 이지만 모든 두개의 화살표를 합성 할 수 는 없다.

- 합성하려는 대상 객체는 소스객체의 화살표가 가르키는 객체와 같아야한다. 프로그래밍에서는 한 함수의 결과와 동일한 타입을 인자로 하는 함수와 합성할 수 있다. → Strong 타입의 언어에서는 이를 더 쉽게 해준다.
- 타입을 다루는것이 프로그래머에게 부담이 될 수 있다. → Haskell 등 대부분 언어에서 타입추론 지원
- 유닛테스팅으로 타입검사를 대체할 순 있지만 컴파일러보다 효과적이지는 못하다.



### 2.3 What Are Types?

프로그래밍에서 타입은 값들의 집합이라 할 수 있다.

- Bool Type은 True, False의 집합, Char type은 유니코드 캐릭터의 집합

- 집합은 유한하거나 무한할 수 있다.

  ```haskell
  x :: Integer
  ```

  -> x는 정수 집합의 원소 중 하나라 할 수 있고 정수집합은 무한하다. (일부 유한한 집합의 집합이 언어에 따라 있다 ex Unit8..)

- 이상적인 세상에서 Haskell의 타입은 집합에, function들은 집합간의 화살표로 나타낼 수 있다. 하지만 수학적인 함수는 어떤 코드도 실행시키지 않는다. 반대로 하스켈 함수는 결과를 계산해야한다.

  - 유한한 범위의, 유한한 연산에서는 문제가 되지 않지만 대상이 매우 크거나 무한 재귀가 걸리는 경우 문제

  - 이를 위해 모든 타입에 "bottom(⊥)"을 추가 → 끝나지 않는 계산을 뜻함, bottom을 타입 시스템에 받아들이면 모든 런타임 에러는 bottom으로 간주 가능하고 함수는 bottom을 반환 할 수 도 있음

    ```haskell
    f :: Bool -> Bool -- may return True, False, or _|_
    f x = undefined   -- undefined를 bottom으로 판단하고 타입 체크 통과
    f = undefined     -- Bool -> Bool 타입에 undefined도 포함되기 때문에 위와같이 나타낼 수 있음 
    ```

    ( bottom때문에 하스켈에서 타입과 함수는 Set보다는 Hask로 언급됨)

그러면 Set의 카테고리는 무엇인가?

- 집합은 구조가 있고 위에서 말한 함수는 집합간 엘리먼트를 맵핑하는 것이라 나타낼 수 있음

- 이를 추상화하여 카테고리로 생각하면 집합 오브젝트와 이들간의 관계인 사상 만 남는다

  → 집합은 내부구조는 들여다볼 필요가 없는 하나의 점이 된다. 결국 한 오브젝트에서 다른 오브젝트로 어떻게 연결되는지(화살표)와 같은 인터페이스만 남겨지게 된다

  

### 2.4 Why Do We Need a Mathematical Model?

프로그램 묘사의 한계점

- 작성한 코드의 의미, 논리를 기술하거나 포멀 형식으로 반환하는 것은 힘들고 한계가 있다. (특히 상용언어) → 하지만 이를 위한 대안인 표시 의미론?(denotational semantics) 이 있으며 수학을 기반으로 한다.

- 표시 의미론에서 모든 프로그래밍 구조에는 수학적 해석이 제공됨 → 수학적 이론을 바탕으로 프로그램을 증명

- 프로그램을 증명하는거는 꽤 쉬운수학..?🤯

- 일반 프로덕션 코드(사이드 이펙트가 있는)에서 denotational semantics은 어떻게 나타내냐?

  - 키보드에서 입력받아서 네트워크 통신하는 로직을 수학적으로 어떻게 나타내야 하나? → 한계가 있음 때론 절차적으로 나타내는것이 더 효과적일 수

    → 카테고리 이론에서는 가능, Eugenio Moggi는 computational effect가 모나드로 매핑될 수 있음을 발견

- 수학적인 증명은 컴퓨터 프로그램의 공식적인 증명을 수행 할 수 있게 해줌



### 2.5 Pure and Dirty Functions

- Dirty Functions: 명령형 프로그램의 함수는 수학적 함수가 아니다, 수학적 함수는 한 값을 다른값으로 매핑하는 것 이다

- Pure Functions: 동일인풋-동일아웃풋, 순수함수는 수학적 모델링으로 쉽게 나타낼 수 있다.

  → 하스켈에서 모두 순수함수, 다른언어는 순수 하위집합으로 제한하거나 부작용에 대해 별도로 다룰 수 있음

  → 추후에 모나드를 이용해 순수함수만을 사용하여 모든 종류의 효과를 모델링하는 방법을 살펴볼것임



### 2.6 Example of Types

**Void Type(Empty Set)**

- empty set은 어떤 타입이냐? → Haskell에서는 Void라 칭함(C++의 void와는 다름)

- Void type을 인자로 하는 함수를 정의는 가능하지만 호출할 방법은 없다 → 그런 이유로 리턴은 모든타입이 될 수 있다 

  ```haskell
  absurd :: Void -> a
  ```

- 논리적으로 Void는 false(거짓됨)를 뜻함

**SingleTon Type(One element Set)**

- 오직 한가지의 값만 나타내는 타입, 값 그자체(이게 오히려 C++의 void를 나타냄)

  → Haskell에서는 이를 () aka Unit 으로 나타냄

- 논리적으로 Unit은 true(옳음)를 뜻함, 그냥 존재하는거고 어떻게든 만들어낼 수 있다.

  ```haskell
  -- haskell에서 ()타입의 엘리먼트는 ()로 다음과 같이 나타낼 수 있다
  () :: ()
  
  
  -- ()을 받아 Int를 반환하는 함수: 순수함수하면 항상 같은값(상수)를 반환할것이다
  One :: () -> Int
  ```

- 유닛을 인자로 받아 어떤 타입으든 반환하는 함수가 존재할 수있다. 만일 리턴타입이 Bool이라면 위와같은 함수는 두개 존재한다(true나 false를 반환)

- 만일 집합의 엘리먼트를 엘리먼트 자체를 이야기 하지 않고 어떻게 정의할 수 있을까? → 싱글톤 셋에서 특정 집합으로 가는 () → a 함수가 있다 생각하면 이는 집합의 엘리먼트들을 피킹(picking)하는 함수이다 → 이를 추후 generalized element of object라 칭할것이다.

- 함수의 리턴타입이 unit인 경우는?

  - c++에서 보통 이런 함수들은 사이드이팩트를 발생한다(ex print) → 하지만 수학적으로는 말이 안되는 함수다

  - 순수함수가 unit을 반환하는것은 아무것도 하지 않는다 → 입력을 무시

  - 수학적으로 위와 같은 경우는 A의 모든 요소를 singleton set으로 맵핑함

    ```haskell
    fInt :: Interger -> ()
    fInt x = ()  -- 뭔 타입을 받던 unit을 리턴
    fInt _ = () -- wildCard pattern
    -- 위의 함수는 인자의 값이나 타입에 의존하지 않는다(parametrically polymorphic)
    
    -- 모든 타입에 대해 확장
    unit :: a -> ()
    unit - = ()
    -- 어떤 타입이든지간에 unit으로의 방향을 지니는 다형성 함수를 unit이라 부르자
    ```

**Bool Type**

- haskell에서 Bool은 빌트인 타입이 아님, Either Ture or False



## 별첨 functions, epimorphism, monomorphism

출처

- [Category Theory 2.1: Functions, epimorphisms](https://www.youtube.com/watch?v=O2lZkr-aAqk&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=3)
- [Category Theory 2.2: Monomorphisms, simple types](https://www.youtube.com/watch?v=NcT7CGPICzo&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=4)



### Function

집합 이론에서 함수는 수학적으로 집합관 관계로 볼 수 있고 관게는 집합의 서브셋간의 pair로 볼 수 있다. 하지만 함수는 **방향성**이 있다

![정의역, 공역, 상](https://lh4.googleusercontent.com/proxy/IL23indZQRgEjf6ktjLr5NuLwtRMiq5TVCsdqrnEd1f1lywTpefPM3ju70LJd_7C2-e7xKa-Qu44SmGS5TJVV4RcG7iXcJUbKe-aXcwnqXypg7VKexbsZHpzA1WNU8oieZkJ=w945-h600-p-k-no-nu)

[출처: https://mathematicspoot.blogspot.com/]

집합 a의 엘리먼트가 f함수를 이용하여 b로 매핑될때 a의 여러 엘리먼트들은 단일 집합 b의 엘리먼트로 맵핑될 수 있다 하지만 반대의경우 하나의 집합 a의 엘리먼트가 여러개의 집합 b의 엘러먼트들로 매핑될 수는 없다. 이런 특성으로 인하여 정의역(Domain)에서 공역(Codomain)을 나타내는 함수의 결과값들의 집합은 공역의 Subset이 되고 이를 상(Image)라 부른다.

#### 만일 특정조건에 역함수가 존재한다면?

```haskell
f :: a -> b
g :: b -> a -- inverse function
g ∘ f = id  -- g와, f의 합성 결과는 f의 인자로 들어왔던 a의 엘리먼트 자체
f ∘ g = id  -- f와, g의 합성 결과는 g의 인자로 들어왔던 b의 엘리먼트 자체
```

→ 반대로 뒤집을 수 있는 경우 함수는 대칭성을 지닌다. ⇒ 이를 isomorphism(동형사상)이라 한다

#### 카테고리의 isomorphism은?

정의역을 a, 공역을 b라 하면 다음과 같이 나타낼 수 있다

```haskell
g ∘ f = ida
f ∘ g = idb
```

__하지만 모든 함수가 isomorphism하지는 않다__

#### 함수가 isomorphism 하기위한 조건

모든 함수는 isomorphism하지 않다

- 이유1: 함수의 결과값이 중복 될 수 있다. 역함수를 생각하면 b의 엘리먼트가 어떤 a로 가야하는지 알 수 없다

  ```haskell
  isEven :: i -> Bool
  -- 자연수를 받아 bool 타입으로 반환하는 함수가 있다고 하면 인자로 입력된 모든 자연수는 true, false 두가지로 귀결된다. 역으로 셍각했을때 true, false를 어떤 수로 맵핑해야하는지 알 수 없다.
  ```

  → 함수는 정의역의 정보를 추상화 한다

- 이유2: 상(Image)은 공역의 subset이다, 정의역을 공역의 일부값으로 매핑한다

  → 모델링

만일 이유1이 만족하지 않는다면(결과가 중복되지 않는다면) 이를 수학적 용어로 injective 하다 한다

만일 이유2가 만족하지 않는다면 이를 subjective 하다 한다

**집합 이론에서 injective하고 subjective한 함수는 isomorphism 하다**

----

### Epimorphism and Monomorphism

카테고리이론에서는 isomorphism을 어떻게 정의? 집합의 세부 구조를 바라보지 않는 카테고리 이론에서는 어떻게 injective하거나 subjective 하다는 것을 정의할 수 있을까?

#### Epimorphism

![epimorphism](https://jeremykun.files.wordpress.com/2013/05/epimorphism-diagram.png)

[사진출처: https://jeremykun.com/tag/epimorphism/]

카테고리에서 subjective는 epimorphism라는 보다 재너럴한 개념으로 나타낼 수 있다.

subjective한 함수 f :: a → b 가 있고 g :: b → c 인 함수가 있다고 할때 f와 g를 결합하는 경우에 대해 생각해보자

subjective하지 않다는 것을 먼저 생각해보자. f가  subjective하지 않다는 것은 공역에서 상을 제거한 영역이 존재한다는 것이다. 이 영역에 존재하는 b1, b2에 대하여 새로운 집합인 C의 동일한 c1값으로 향하는 다음의 두 함수가 있다고 하자

```haskell
g1 b1 = c1
g2 b2 = c1
```

수학적으로 g1과 g2는 인자가 다르기 때문에 다른 함수이다. 마찬가지로 다음의 경우에도 g1, g2는 다르다고 할 수 있다(결과값이 다르기 때문)

```haskell
g1 b1 = c1
g2 b1 = c2
```

하지만 f와의 합성 관계를 생각할때 두 함수의 시작점이 모두 f의 상 외부에 존재한다는 점은 공통이다. f와 g의 합성은 f의 상 내에서만 작동할것이다. f의 외부 상을 인자로하는 함수 두개 g1, g2를 취하면 두개의 합성 g1∘ f, g2 ∘ f는 결국 동일하고 다른 g1, g2에도 불구하고 다음의 관계를 지닐 수 있다

```haskell
g1 o f = g2 o f 
```

이를 반대로 생각하면 epimorphism을 정의할 수 있다.

모든 c에 대하여 f는 epimorphism이고 g1 o f = g2 o f는 g1 = g2를 의미한다.~~(이해 못함...)~~ 



#### Monomorphism

![Monomorphism](https://jeremykun.files.wordpress.com/2013/05/monomorphism-diagram.png?w=448&h=108)

[사진출처: https://jeremykun.com/tag/epimorphism/]

카테고리에서 injective는 monomorphism이라는 재너럴한 개념으로 나타낼 수 있다.

마찬가지로 injective 하지 않은 상태를 가정해서 생각해보자 injective 하지 않은 다음의 두 함수가 있다 가정하자 f1 :: a1 → b1, f2 :: a2 → b1. 그리고 임의의 c에 대하여 g :: c → a 인 함수 g가 존재한다고 할때 g와 f1, f2의 합성의 결과값은 동일할 것이고 이경우 injective하지 않다

이를 또 반대로 생각하면 monomorphism을 다음과 같이 정의할 수 있다.

모든 c에 대하여 c → a인 함수 g1, g2가 있다고 할때 둘의 합성 g1 . f = g2 . f 이며 g1 = g2



그럼 카테고리에서 두 성질을 만족할때 isomorphism이 성립할까 → No 🤯

