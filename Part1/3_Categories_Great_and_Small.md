## 3. Categories Great and Small

지난시간에 집합과 집합의 카테고리에 대하여 알아봤다.(특히 집합적 관점에서 집합의 엘리먼트에 집중했었다.) set의 카테고리 말고도 무수히 많은 일반 카테고리 중 몇가지 예시를 알아볼 것이다.



### 3.1 No Objects

: 대상이 없고 사상도 없는 카테고리, 가장 간단한 형태의 카테고리

- 대상이 없기때문에 사상도 없다

- 이경우 카테고리의 공리가 자연스럽게 만족됨

  - 사상의 합성에 관해서 사상자체가 없으면 자연스럽게 조건을 만족한다
  - 대상의 항등사상이 있어야한다 → 오브젝트 자체가 없어서 자동으로 만족

- empty 카테고리는 쓸모가 없지만 다른 카테고리와의 합성 맥락(context)상에서 유의미하다. 모든 카테고리의 카테고리, small category에서 이것은 중요합니다

  

### 3.1.1 One Object

: 하나의 대상만을 지니는 카테고리

- 하나의 대상만 존재하므로 고로 하나의 사상(항등사상)만 존재한다.
- 마찬가지로 다른 카테고리와의 맥락상에서 중요하다.



### 3.2 Simple Graphs

만일 두개의 대상을 지니는 카테고리가 있다고 하면 대상 각자의 항등 사상과, 대상간의 사상 등등 이 있다고 할 수 있다. 우리는 이를 그래프로 나타낼 수 있다.

![](https://larzac.info/maths/categories/milewski/younotes1/img/graph2category.jpg)

- free graph

  - 항등사상, 대상을 연결하는 사상, 이를 합성하는 사상 등 무수히 많은 사상을 그릴 수 있다. (카테고리의 공리를 만족시키기 위해 결합법칙을 고려하며)

  - 유한 그래프의 경우 유한개의 사상으로 카테고리를 형성할 수 있다.

    → 카테고리의 공리를 만족시키며 그릴 수 있는 모든 사상, 그래프를  free graph라 한다.

  

### 3.3 Orders

: 카테고리에서 사상은 대상간의 관계를 나타낸다. <= 와 같은 관계가 카테고리를 만족하는지 알아보자

- 항등 사상(Identity Morphism)이 존재하는가?: 모든 대상이 자기 자신보다 작거나 같다는 조건을 만족한다 -> okay

- 관계(사상을) 합성 할 수 있는가?: 예를들어 대상 a, b, c에 대하여 a <= b, b <= c이라면 a <= c 이다 -> okay

- 결합법칙도 자연스럽게 만족한다.

- Preorder(원순서)

  - 집합에서 위와같은 관계를 preorder라 한다. 그리고 preorder는 카테고리이다

- Partial Order(부분 순서)

  - a <= b, b <= a 인 관계에서 a = b 인 관계가 존재한다

  - 원순서에 위와 같은  강한 관계를 부여하는경우 이를paritial order라 한다

  - partial order의 카테고리에서 a -> b이 있을때 b -> a와 같은 역의 관계는 허용하지 않는다

    ( partial order를 예로들어 Epicmorphism하고 isomorphism한 모든 사상이 isomorphism 하지는 않다 말할 수 있다. -> 역관계가 없어서)

- Total Order(전순서)

  - 두 객체가 어떤식으로든 서로 관계를 맺고 있다는 조건이 부과된 상태

 - Thin category
    - preorder는 두 대상간에 사상이 없거나 1개만 존재하는 경우다 -> 위와같은 카테고리를 thin category라 부른다(많으면 thick category)
       - 범주 C의 대상 a에서  대상 b로 향하는 사상의 집합을 home-set이라 부르고 다음과 같이 쓴다: C(a, b) (or Hom𝐂(𝑎, 𝑏))
   - 따라서 모든 preorder의 홈셋은 비어있거나 singleton이다.(a -> b가 없거나 1개 이므로)
   - thin category와 preorder는 1:1 대응 관계이다

### 3.4 Monoid as Set

집합 이론의 관점에서 모노이드는 어떻게 정의될까?

- 전통적으로 모노이드는 이진연산(결합법칙 만족)을 사용하는 집합으로 정의된다.

- 또한 unit 역할을 하는 특별한 원소가 있다

- ex) 자연수 집합에 덧셈 → 0이 unit 엘리먼트 역할을 함

  ```haskell
  0 + a = a
  a + 0 = a  -- 덧셈은 교환법칙이 성립하지망ㄴ 교환법칙이 모노이드의 성질은 아니다
  
  -- 곱셉을 생각하면 1이 unit 엘리먼트 역할을 함
  ```

- Monoid in Haskell

  ```haskell
  class Monoid m where
  	mempty :: m   -- neutral element
  	mappend :: m -> m -> m -- binary operation
  	-- m을 인자로 받아 함수를 리턴한다는 의미로 다음과 같이 나타낼 수 있음 mappend :: m -> (m -> m)
  	
  instance Monoid String where
  	mempty = ""
  	mappend = (++)
  ```

- Monoid in Swift

  ```swift
  protocol Monoid {
      associatedtype M
      var mempty: M { get }
      func mappend(_ m1: M, _ m2: M) -> M
  }
  
  extension String: Monoid  {
      
      typealias M = String
      
      var mempty: String { return "" }
  
      func mappend(_ m1: String, _ m2: String) -> String {
          return m1 + m2
      }
  }
  ```

### 3.5 Monoid as Category

![](https://larzac.info/maths/categories/milewski/younotes1/img/one-object-category.jpg)

앞서 정의한 집합의 모노이드와 다르게 카테고리에서 생각할때는 우리는 집합이나 그들의 엘리먼트에 대해서 생각하지 않는다. 대신 우리는 대상과 화살표에 집중한다. 카테고리 이론에서 집합 모노이드의 이항연산을 원소를 moving" 혹은 shifting" 한다로 바꿔 생각해보자

```haskell
-- 정수에 5를 더하는 연산 함수(adder)에 대해 예
-- 0 -> 5: 0을 5로 맵핑
-- 1 -> 6: 1을 6으로 맵핑
-- => adder 함수는 정수 집합에 정의되어있다.
```

- adder 함수를 어떻게 합성할까? adder 5와 adder 7의 합성은 12를 더하는 것 이라 말할 수 있다

  → adder 함수의 합성은 덧셈연산과 동일하다 ⇒ 덧셈을 함수 합성으로 대체할 수 있다.

- 0을 더하는것은 뭘까? ⇒ 단위 사상을 뜻한다.

- 자연스럽게 adder함수의 합성은 결합법칙이 성립한다.

> 기민한 독자라면 위에서 나왔던 mappend :: m -> (m -> m)의 타입 시그니처에 대해 눈치를 챘을수있다. (예를들어 정수를 adder로 매핑) mappend가 모노이드 집합의 원소를 받아 집합에서 작동하는 함수로 매핑하는것을 뜻합니다.

![](https://bartoszmilewski.files.wordpress.com/2014/12/monoidhomset.jpg?w=300&h=197)

- Monoid는 하나의 대상만을 지니는 카테고리이다. 모든 모노이드는 하나의 대상과 적절한 합성 규칙을 따르는 사상들의 집합으로 구성되어있다 묘사 할 수 있습니다.

  -> 모노이드 집합 m의 사상들은 카테고리 모노이드 M에서 hom-set의 원소에 해당한다 => 1:1 대응하는 관계에 있다.

