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