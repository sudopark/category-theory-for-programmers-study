## 4. Kleisli Categories

타입과 순수함수를 카테고리에 모델링하는 방법을 알아보았습니다. 또한 저자는 사이드 이펙트나 순수하지 않은 함수를 모델링 하는 방식이 있다 일전에 언급했었습니다. 이번에는 어떤 함수의 동작을 로깅하는 함수를 예를들어 봅시다.

명령형 프로그래밍에서 전역변수를 이용하여 위의 함수를 아래와 같이 구현할 수 있습니다.

```swift
var logger: String = ""

func negate(b: Bool) {
  self.logger != "Not so! "
  return !b
}
```

위 함수는 사이드이펙트를 발생시키고 순수함수가 아닙니다. 함수의 인자와 리턴값에 로그값을 주어 전역변수 없이 다음과 같이 순수함수로 구현할 수도 있습니다. 하지만 누적으로 로그값을 유지하려면 지속적으로 로그 히스토리를 들고있어야하고 인터페이스도 별로 안좋아집니다.

```swift
func negate(b: Bool, logger: String) {
  return (!b, logger + "Not so! ")
}
```

위 함수의 결과값은 용도에 따라 로그값은 무시할수도 있지만 인자에 로그는 무조건 입력해줘야합니다. negate 함수의 주된 목적은 입력받은 bool값을 인버팅하는것이고 로깅하는 동작은 부가적입니다. 특히나 로그의 경우 다른 두 함수의 콜사이에 누적되어야합니다. 

입력받은 문자를 대문자로 변환하는 함수와 단어로 나누어 [String]을 반환하는 두 함수가 있을때 각각의 함수에 로그기능을 추가하고 두개를 합치는 경우를 생각해봅시다. 이번에는 Writer라는 제네릭 tupple을 이용하여 리턴값을 원하는 타입과 로그를 래핑할겁니다.

```swift
typealias Writer<A> = (A, String)
func toUpper(s: String) -> Writer<String> {
  return (s.uppercased(), "toUpper ")
}

func toWords(s: String) -> Writer<[String]> {
  return (s.components(separatedBy: " "), "toWords ")
}

// toUpper + toWords
func process(s: String) -> Writer<[String]> {
  let pair1 = toUpper(s)
  let pair2 = toWords(pair1.0)
  return (pair2.0, pair1.1 + pair2.1)
}
```

위와같이 합성을 이용해서 이제 누적되는 로그의 집계는 개별 함수의 문제가 아니게 됩니다. 그럼 이제 모든 프로그램을 이렇게 바꿔보자! -> 완전 악몽일것입니다. 물론 우리는 추상화를 통하여 이를 해결할 수 있지만 특히 함수의 합성을 추상화 해야합니다. 추상화를 시작하기 이전에 카테고리적 관점에서 문제를 분석해봅시다.

### 4.1 The Writer Category

타입과 함수에 대한 카테고리에서부터 시작하자. 타입은 대상으로 남겨두고 사상을 embellished functions으로 재정의 합시다.

int 값을받아 짝수인지 판단하여 bool로 반환하는 isEven 함수가 있을때 이를 부가정보를 더해 리턴값으로 반환하는 함수로 바꾼다고 생각해보자. 이경우 리턴값에 bool외 추가적인 정보가 페어로 추가된다 하더라도 중요한점은 사상은 여전히 int -> bool 이라는 관계를 지니고 있다 간주된다는 점입니다.

```swift
func isEven(n: Int) -> (Bool, String) {
  return (n % 2 == 0, "isEven ")
}
```

카테고리의 특성에 따라 우리는 위 함수를 bool을 입력으로 받는 함수와 합성할 수 있어야합니다. 위에서 예를들었던 negate와 합쳐 isOdd를 만드려할때, 당연히 in/out이 매칭되지 않아 안될것입니다. 합성은 다음과 같은 모습 이여야할것입니다.

```swift
func isOdd(n: Int) -> (Bool, String) {
  let pair1 = isEven(n: n)
  let pair2 = negate(b: pair1.0)
  return (pair2.0, pair1.1 + pair2.1)
}
```

자 우리가 만들고있는 이 카테고리에서 합성을 위해서 필요한 레시피는 다음과 같습니다.

1. 첫번째 사상에 해당하는 embellished function을 수행한다.
2. 결과값으로 나온 페어에서 첫번째값을 빼내고 두번째 사상에 해당하는 함수에 입력한다.
3. 첫번째 결과 페어와 두번째 결과 페어의 두번째값(string)을 합친다.
4. 새로운 페어를 반환한다(두번째 결과값, 첫번째 결과값의 String + 두번째 결과값의 String)

다시 위의 toUpper와 toWords를 합치는 문제로 돌아와서 고차함수를 이용하여 합성을 추상화 해봅시다.

```swift
func process<A, B, C>(m1: @escaping (A) -> Writer<B>, 
                      m2: @escaping (B) -> Writer<C>) -> A -> Writer<C> {
  return { a in
    let p1 = m1(a)
    let p2 = m2(p1.0)
    return (p2.0, p1.1 + p2.1)
  }
}
```

아직 끝난것은 아니다. 위는 항등 사상은 아니다. 인자를 받아 그대로 반환하는 A -> Writer<A> 형식의 항등 사상이 필요합니다. 

```swift
func identity<A>(a: A) -> Writer<A> {
  return (a, "")
}
```

방금 정의한 카테고리는 카테고리의 요구사항을 충족시키는 카테고리입니다. 그리고 물론 결합법칙도 따(합성의 페어의 첫번쨰 인자를 합성하는 방식을 보면 결합법칙을 따르는 일반적인 함수 합성임을 알 수 있다. 페어의 두번째값 역시 String을 누적하는것이고 이는 당연히 결합법칙이 성립한다.) **기민한 독자라면 이 구조를 String monoid 외에 일반화된 monoid로 확장할 수 있음을 눈치챘을것입니다.** 위의 compose 함수에서 mappend를 identity 함수에서는 mempty를 수행하고있는 것입니다.

### 4.2 Writer in Haskell

Writer를 하스켈에서 구현해보자, 우선 Writer 타입을 정의할것입니다.

```haskell
type Writer a = (a, String)
```

어떤 타입에서 어떤 타입의 Writer를 반환하는 사상은 다음과 같이 나태고 "fish"라고 불리는 인픽스 연산자를 이용하여 합성을 선언하고 정의할것입니다.

```haskell
-- morphism
a -> Writer b

-- composition
(>==>) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)

-- the result is a lamda function of one argument x
m1 >=> m2 = \x -> 
  let (y, s1) = m1 x
      (z, s2) = m2 y
  in (z, s1 ++ s2)
```

그리고 return 이라는 이름의 항등사상을 다음과같이 정의할것입니다.

```haskell
return :: a -> Writer a
return x = (x, "")
```

그럼 이제 위에서 했던것처럼 toUpper와 toWords를 합성해보자

```haskell
upCase :: String -> Writer String
upCase s = (map toUpper s, "upCase ")
-- map은 character의 toUpper함수를 string s에 적용합니다.

toWords :: String Writer [String]
toWords s = (words s, "toWords ")
-- words는 표준 라이브러리 prelude에 정의된 함수입니다.

process :: String -> Writer [String]
process = upCase >=> toWords

```



### 4.3 Kleisli Categories

![kleisli category](https://larzac.info/maths/categories/milewski/younotes1/img/kleisli.jpg)

​     [출처: https://larzac.info/maths/categories/milewski/younotes1/4.1-terminal-and-initial-objects.html]

우리는 새로운 카테고리를 만든것이 아닙니다. 이것은 모나드를 기반으로 하는 카테고리인 kleisli 카테고리의 한 예 입니다.(모나드는 여기서 맛보기만 하고 나중에 알아봅시다) 클라이슬리 카테고리에서는 프로그래밍에서의 타입이 대상에 포함됩니다. **그리고 A타입에서 B 타입으로의 사상은 A에서 특정 장식(emblishment)를 적용한 B로의 함수에 대응됩니다.모든 클라이슬리 카테고리는 이런 사상들을 합성하는 각자의 방식이 있습니다.** 또한 이 합성법의 항등사상도 있습니다. (추후에 우리는 emblishment라는 용어가 카테고리에서 "endofunctor"라는 개념에 대응한다는 것을 알아볼것입니다.)

위에서는 함수의 실행을 로깅하기 위한 Writer라는 모나드를 이용했지만 이는 순수한 함수에 효과를 포함시키기 위한 일반적인 메커니즘의 예시 이기도 합니다. 이전에는 우리는 프로그래밍의 타입을 집합의 카테고리로 모델링 하는 법에 대하여 알아봤습니다(bottom이라는 개념은 제외하고). 이번에 한것은 모델을 약간 다른 카테고리로 확장했습니다, 그리고 이 카테고리에서 사상은 emblished function에 해당하고 이들의 합성은 단지 리턴값을 다른 함수에 인자로 넣는것 이상이였습니다. 우리는 합성 자체를 가지고 노는데 자유도?가 한단계 늘었습니다. 이것은 전통적인 명령형 언어 프로그램에서의 사이드 이펙트를 간단한 denotational 의미로 나타낼 수 있는 자유도 입니다.

## 별첨 Terminal and initial objects

출처

- [Category Theory 4.1: Terminal and initial objects](https://www.youtube.com/watch?v=zer1aFgj4aU&list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&index=7)
- [4.1: Terminal and initial objects](https://larzac.info/maths/categories/milewski/younotes1/4.1-terminal-and-initial-objects.html)

우리는 집합에 대하여, 집합의 카테고리에 대하여 그리고 집합이론에 대해서도 이야기 했었다. 두가지 이론을 이용하여 바라보는 시선은 매우 중요하다. 집합은 원소를 포함합니다. 그리고 우리는 이 원소를 다른원소로 매핑하는 함수를 정의할 수 있습니다. 또한 원소가 없는 empty set도 있고 원소가 하나만 있는 singleton set도 있습니다. 이 모든것은 집합에서 원소를 기준으로 정의되었습니다.

이후 우리는 집합의 카테고리에 대하여 이야기 하기 시작하였고, 집합이 소유하는 원소와 같은 디테일에서 멀어졌습니다. 원소에 대하여 이야기하는 것 대신에 카테고리에서 우리가 집중한것은 화살표였습니다. 집합 이론에서 이 화살표는 원소들간의 함수에서 시작했습니다. 집합의 원소간 함수가있을때 이것은 집합의 카테고리에서 화살표가 되었습니다. 물론 우리는 이 함수들을 어떻게 합성해야하는지 압니다. 먼저 집합적인 관점에서 이 함수들을 어떻게 합성할것인지에대하여 알아보고 카테고리적 관점으로 넘어와 원소에대한 디테일은 잊어버리고 합성된 함수가 새로운 화살표를 만든다는것을 보았습니다.

Empty set은 카테고리에서 무엇일까요? 원소가 없다는 것은 무엇일까요?(카테고리는 원소를 모르는데) 원소를 모르는데 empty set이나 singleton set을 어떻게 정의할까요? 원소를 모르는데 두 집합간의 카티시안곱(cartesian product)를 정의할까요? 우리는 이 모든것을 재발견하고 재정의해야합니다. 오직 화살표의 합성 측면에서. 

### Universal Construction

이를 위해 아주 일반화된 방법이 있는데 이를 보편성질(Universal construction)라 합니다. 보편적 성질에서는 어떤 대상이나 화살표, 패턴을 고르고 대상의 성질을 다른 대상간의 관계로 정의합니다. 그리고 이 관계는 화살표를 뜻합니다. 그래서 우리는 화살표적인 관점에서 모든것을 재정의 해야합니다. 그것은 한 대상에서 다른 대상으로의 들어오는(incomming) 나가는(outgoing) 화살표 일수도 있습니다. 카테고리 내에 한 대상에 대해서 다른 모든 대상과의 관계를 정의하고싶다면 전체에 대하여 생각해야합니다.

보편적 성질은 마치 구글링 하는것과 같습니다. 당신은 어떤 특정 패턴을 구굴에게 매칭되는 모든것을 보여달라 요청합니다. 검색한 패턴이 매우 보편적인 패턴이라면 무수히 많은 혹은 무한대의 결과값이 나올 수 있습니다. 그리고 당신은 검색결과에 랭킹을 부여합니다. 이것은 동일한 패턴을 만족하는 결과중에서 어떤것이 더 낫다, 비교할 수 있음을 뜻합니다. 일전에 보았던 preorder에서는 모든것을 비교할수는 없지만 일부는 비교할 수 있음을 보았습니다. 그리고 이런 랭킹들의 최상위에는 우리가 찾으려고하는 최적의 결과가 있습니다.

### Terminal Object

원소가 하나인 singleton set으로 돌아와 원소의 디테일에 대한 이야기 없이 우리는 어떻게 이 집합이 다른 집합과 관계를 맺는다 말할 수 있을까요? 싱글톤 셋의 흥미로운점은 다른 모든 종류의 집합에서 오는 화살표가 있다는 점 입니다.

![](https://larzac.info/maths/categories/milewski/younotes1/img/singleton-incoming.jpg)

 어떤 타입이나 어떤 종류의 집합에서 유닛(싱글톤셋) 으로 향하는 화살표가 있습니다. 이는 다형성이 있고 이는 아규먼트를 무시하고 유닛을 반환하기 때문에 유닛이라 불를 수 있습니다. 그리고 또한 Void(empty set)에서 유닛으로 가는 화살표도 있습니다. 이것은 카테고리에서 모든 대상에서 부터 오는 화살표가 있다는 싱글톤 셋의 보편적인 성질이다. 집합간에는 무수히 많은 화살표들이 있을 수 있다. 하지만 비어있지 않은 집합에서 빈 집합으로 향하는 화살표는 없다.(어떤 집합의 원소가 빈 집합으로 향할 수 없기 때문에) 종착지가 엠티셋인 경우를 제외하고 모든 집합들은 다른 집합으로 향할 수 있는 화살표가 있다.  

만일 어떤 타입의 집합에서 bool 집합으로 향하는 화살표가 있다고하자, 그러면 적어도 2개의 화살표가 존재할것이다(true로 가거나, false로 가거나) 그러면 어떤 대상에서 종착지가 싱글톤 셋으로 가는 경우에는 **유일한** 화살표가 1개 있다 할수있다. 그럼 이런 성질들을 이용하여 카테고리에서 싱글톤셋은 무엇이라 말할 수 있을까? 이와같은 대상을 뭐라할까? 이를 terminal object라 한다. 모든 카테고리가 terminal object를 지니지는 않는다 하지만 모든 카테고리에 대하여 termial object를 정의할 수 있다. 카테고리에서 터미널 오브젝트는 어떤 대상으로부터 유일한 화살표를 지니는 대상이고 두개의 조건이 있다.

- 조건1: 모든 대상에 대하여 터미널 오브젝트로 향하는 화살표가 있어야한다
- 조건2: 그리고 터미널 오브젝트로 향하는 화살표(함수)f, g가 있을때 둘은 같아야한다.

thin category와 order를 이용해 예를 들어보자. 이들은 항상 터미널 오브젝트를 지니는가? order에 있어 a -> b인 화살표가 있다면 이는 a <= b 임을 뜻한다. 만일 모든 대상에서 터미널 오브젝트로 향한다는것은 모든 대상이 터미널오브젝트보다 작거나 같음을 뜻한다. 그러면 터미널 오브젝트는 이 카테고리에서 가장큰 오브젝트가 될것이다.

### Initial Object

![](https://larzac.info/maths/categories/milewski/younotes1/img/absurd.jpg)

그러면 위에서 말한 화살표를 뒤집어보자. 싱글톤셋과 반대로 엠티셋의 속성은 모든 대상으로 나가는 화살표로 정의할 수 있다.(앞서 이 함수는 absurd라 이름지었다). 터미널 오브젝트의 정의를 뒤집어보자. 카테고리에서 어떤 대상으로든 갈 수 있는 대상은 무엇일까? 마찬가지로 이 화살표도 유일해야한다. 이를 Initial Object라 부른다. 카테고리에서 이니셜 오브젝트는 어떤 대상으로든 향하는 유일한 화살표가 있는 대상이다. 그리고 이는 집합의 엠티셋에 해당한다.

### Unique Path

카테고리에 이니셜, 터미널 오브젝트가 있을때(무조건 있어야 하지는 않지만) 카테고리에서 어떤 대상을 선택하고 어떤 경로를 거쳐서든 터미널 오브젝트로 향하는 화살표를 그린다면 이를 모두 합성해서 바로 터미널 오브젝트로 향하는 하나의 화살표로 대체할 수 있다. 터미널 오브젝트의 속성에 따라 어떤 종류의 경로를 거쳐서라도 터미널오브젝트로 향하는 화살표들이 있다면 이를 하나의 화살표로 줄일 수 있다. 그리고 이는 같다, 유일하다. 

### Uniqueness

다음 질문으로 그럼 이런 오브젝트들은 얼마나 있을까? 얼마나 많은 엠티셋이 있을까? 1개이다. 터미널은? 싱글톤셋은? 카테고리적 관점에서 두 대상이 같다는 것은 무엇을 뜻할까? 우리는 모른다. 카테고리에서 우리는 모른다, 카테고리에서 대상간 동일하다는(equality) 개념은 없다. 하지만 화살표가 같은지 따질수는 있다.(그들의 시작점과 종착점이 같냐는 점으로 - 근데 시작, 종착점에 해당하는 대상을 비교못하면 화살표는 어떻게 구분..?) eqaulity를 정의하는것보다 우리는 카테고리에서 isomorphism을 정의하는게 쉽다. isomorphism은 f :: a -> b인 화살표가 있을때 이의 역에 해당하는 g :: b -> a가 있음을 뜻한다.

터미널 오브젝트에 대하여 생각해보자. 터미널 오브젝트는 isomorphism의 관점에서 유일하다. 두 터미널 오브젝트는 isomorphism을 만족한다. 그리고 그둘간에는 유일한 isomorphism이 존재한다