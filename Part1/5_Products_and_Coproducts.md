## 5. Products and Coproducts

카테고리에서 특정 대상을 구분하려고 하려면 다른 대상과의 관계 패턴(혹은 자기 자신과의 관계)을 설명하여야합니다. 이와같은 관계를 정의하기 위해서 카테고리에는 universal contruction이라 불리는 일반적인 확장법(Contruction)?이 있습니다. 이것을 하는 방식은 대상이나 사상을 이용하여 한 패턴을 골라 카테고리내에서 모든 존재를 찾는것 입니다. 만일 이 패턴이 범용적이고 카테고리가 크다면 매칭되는 결과는 매우 많을것입니다. 그리고 난 이후에 결과물사이에서 랭킹을 적용하여 최적의 매칭값을 찾습니다.

보편적 성질은 마치 구글링 하는것과 같습니다. 당신은 어떤 특정 패턴을 구굴에게 매칭되는 모든것을 보여달라 요청합니다. 검색한 패턴이 매우 보편적인 패턴이라면 무수히 많은 혹은 무한대의 결과값이 나올 수 있습니다. 그리고 당신은 검색결과에 랭킹을 부여합니다. 이것은 동일한 패턴을 만족하는 결과중에서 어떤것이 더 낫다, 비교할 수 있음을 뜻합니다. 그리고 이런 랭킹들의 최상위에는 우리가 찾으려고하는 최적의 결과가 있습니다.

### 5.1 Initial Object

주어진 카테고리에서 제일 단순한 형태는 대상 그 자체 일 것입니다. 그리고 이에 매칭되는 결과값은 매우 많습니다. 그리면 이 대상중에서 최상위 객체를 찾기위해 랭킹을 부여해 봅시다. 우리가 할수있는 유일한 방법은 사상입니다. 사상을 화살표와 같이 생각한다면 카테고리 내에서 대상간에 무수히 많은 화살표가 가능함을 상상할 수 있을것 입니다. 카테고리가 이는 partial order와 같이 정렬되었다고 생각해볼 수 있습니다. 우리는 이와같은 관계에서 어떤 대상 a, b에 대하여 a -> b로의 사상이 있다면 a가 b보다 더 initial 한 상태가 있다 일반화 할 수 있습니다.(물론 위와같은 대상이 존재한다 장담할수는 없지만 일단 넘어갑시다.) 문제는 위와같은 대상들이 무수히 많을 수 있다는 점 입니다. 두 대상 사이에 오직 하나의 화살표가 있다는 가정하에 카테고리가 정렬되어있다는 점에서 힌트를 얻을 수 있습니다. 그리고 이 사실은 시작대상(initial object)의 정의를 이끕니다.

**카테고리에서 시작대상은 카테고리의 모든 대상으로 이동하는 유일한 사상이 존재하는 대상입니다.** 하지만 이는 시작대상(만일 존재한다면)의 고유함을 보장하지는 않습니다. 그러나 동형사상(isomorphism)에 따라 고유성을 보장할 수 있습니다. isomorphism은 카테고리 이론에서 매우 중요합니다. 일단은 isomorphism에 따라 시작대상의 고유함을 정의할 수 있다하고 넘어갑시다.

집합과 함수의 카테고리에서 시작대상은 빈 집합(empty set)에 해당합니다. 엠티셋은 하스켈에서는 Void타입에 해당하고 일전에 우리는 Void에서 어떤 타입 로 가는 다형성을 지니는 함수를 absurd라는 이름으로 정의했었습니다.

```haskell
absurd :: Void -> a
```

Void를 타입의 카테고리에서 이니셜 오브젝트로 만드는 것은 바로 이와같은 계열의 사상입니다.



### 5.2 Terminal Object

이제는 하나의 대상만 있는 패턴을 생각해봅시다. 하지만 오브젝트에 랭킹을 먹이는 방식을 좀 다르게 해봅시다. 대상 a, b에 대하여 b -> a인 사상이 있다 할때 a가 "more termial" 하다 해봅시다(inital과 반대방향). 우리는 카테고리에서 어떤 대상들보다 더 terminal한 대상을 찾고있습니다. 또한 고유한것도 포함해서요.

**카테고리에서 터미널 오브젝트는 모든 대상으로부터 오늘 유일한 사상이 있는 대상입니다.** 역시 터미널 오브젝트는 isomorphism의 관점에서 유일합니다. 우리는 일전에 싱글톤에 대하여 말하였고 이는 하스켈에서 unit type ()에 해당합니다. 또한 unit type으로 귀결되는 순수함수는 오직 하나라는 점을 보았습니다. -> 터미널 오브젝트의 모든 조건을 만족

```haskell
unit :: a -> ()
unit _ = ()
```

이 예에서 고유함의 조건은 매우 중요합니다. 특히나 엠티셋을 제외한 모든 집합에서 유닛으로 향하는 사상에 대해서 말입니다. 종착 집합이 Bool인 경우 예를 들어보겠습니다.

```haskell
yes :: a -> Bool
yes _ = True
no :: a -> Bool
no _ = False
```

위에서 보듯이 Bool로 향하는 화살표는 1개 이상입니다. 그러므로 Bool은 터미널 오브젝트가 아닙니다.

고유성에따라 터미널 객체의 정의는 하나의 타입이라 좁힐 수 있습니다.(Insisting on uniqueness gives us just the right precision to narrow down the definition of the terminal object to just one type.)

### 5.3 Duality

Initial Object와 Terminal Object간에 대칭성이 있다는것을 눈치챘을겁니다. 두 오브젝트간 다른점은 방향입니다. 모든 범주 C에 대하여 모든 화살표를 반대로하면 반대범주 𝐂𝑜𝑝를 얻을 수있습니다. 그리고 우리가 합성을 다시 정의할 수 있다면 반대범주는 자동으로 카테고리의 조건을 만족시킵니다. 원래 카테고리에 다음과 같은 사상과 사상의 합성이 있다면 우리는 반대범주에서 다음과 같이 뒤집을 수 있습니다.(그리고 identity 사상을 뒤집는것은 아무소용이없습니다)

```haskell
f :: a -> b
g :: b -> c
h :: a -> c
h = g ∘ f

-- reverse
fop :: b -> a
gop :: c -> b
hop :: c -> a
hop  = fop ∘ gop
```

카테고리 이론에서 듀얼리티는 매우 중요합니다. 듀얼리티를 이용하면 수학적 작업의 생산성을 높일 수 있습니다. 듀얼리티를 통해 어떤 일반화를 증명한다면 반대의 경우도 거저로 얻을 수 있기 때문입니다. (보통 반대의 경우 co 프리픽스를 붙여 부릅니다., products & coproducts, monads & comonads, cones & cocones, limit & colimit 등등..) 하지만 화살표를 두번 뒤집으면 원래상태로 돌아가기 때문에 cocomonads 같은것은 없습니다.

이에따라 카테고리에서 terminal object는 initial object의 반대가 됩니다.



### 5.4 Isomorphism

프로그래머로서 동등함(equality)를 증명하는것은 쉽지 않다는 것을 알겁니다. 두 오브젝트가 같다는 것은 무엇을 의미합니까? 두 객체가 동일한 메모리 공간에 위치함을 뜻합니까? 아니면 구성 요소의 값이 같은 경우입니까? 컴플렉스 넘버에 대하여 하나는 실수부와 허수부를 표현하고 다른 하나는 계수와 각도를 표현한다면 이둘은 같은겁니까? 당신은 수학자들이 동등의 의미를 알아냈을것이라 생각하지만 그렇지 않습니다. 그들도 마찬가지로 동등을 정의하는데 어려움이 있습니다. equality 종류도 많습니다(생략) 그리고 동형(isomorphism)에 대한 더 약한(weaker) 개념과 동등성(equivalence)에 대한 더 약한 개념이 있습니다.

직관적으로 동형을 이루는 객체들은 동일해보입니다. 그들은 같은 형태를 이루고 있습니다. -> 이는 어떤 객체의 한 부분이 다른 객체의 다른 부분으로의 매핑에서 1:1 관계를 지니고 있음을 의미합니다. 우리가 묘사할 수 있는한 두 물체는 서로 완벽한 사본입니다. 수학적으로 객체 a, b가 있고 a -> b인 매핑이 있다면 역으로  b -> a인 매핑또한 존재하고 서로 역의 관계이다를 의미합니다. 카테고리이론에서 우리는 매핑을 사상으로 대체했습니다. 동형 사상은 뒤집을 수 있는 사상을 말합니다. 혹은 한 사상과 역사상의 쌍을 의미합니다.

합성과 identity의 역에 대하여 생각해봅시다. 어떤 사상 f에 대하여 역사상 g가 있을때 두 사상의 합성은 g.f가 항등사상이 됩니다.

```haskell
f . g = id
g . f = id
```

위에서 이니셜 오브젝트나 터미널 오브젝트는 동형사상 측면에서 유일하다고 하였습니다. 이것은 이 객체들간에(이니셜 터미널 각각) isomorphism 함을 나타냅니다. 예를 들어봅시다. 두 이니셜 오브젝트 i1, i2가 있다고 할때 i1은 이니셜 오브젝트 이기때문에 f :: i1 -> i2인 사상은 유일할것입니다. 똑같은 이유로 g :: i2 -> i1의 사상도 유일합니다. 그러면 두 사상의 합성은 무엇입니까?

사상의 합성 g . f은 i1 -> i1을 뜻합니다. 하지만 i1은 이니셜 오브젝트 이기때문에 i1 -> i1인 사상은 오직 1개 입니다.(이는 identity 사상과 일치합니다) g . f는 i1의 항등사상과 동일합니다. 반대의 경우 f . g는 i2의 항등사상과 동일합니다. 이는 f와 g가 서로 역관계에 있음을 증명하고 두 시작대상은 isomoriphic 하다 할 수 있습니다.

시작대상에서 시작대상으로 향하는 사상(항등사상)은 유일하다는 점을 이용하여 동형사상에 이르는 부분을 증명했습니다. 근데 우리는 왜 f, g의 고유함이 필요할까요? 동형사상에 따라 시작대상이 고유하다는 것은 물론 고유한 동형사상에 따라 고유함 때문입니다. 여기서 다루진 않겠지만 원칙적으로 대상간 하나 이상의 동형사상이 존재할 수 있습니다. 이 "고유한 동형에 이르는 고유성"은 모든 universal construction에 중요한 속성입니다.



### 5.5 Products

다음 universal construction은 product이다. 우리는 두 집합간의 곱집합(Cartesian product)를 안다. 그것은 pair의 집합니다. 그러면 곱집과 이를 구성하는 집합을 연결하는 패턴은 무엇일까? 이점을 알아낼 수 있다면 이를 다른 카테고리에 일반화 할 수 있을것이다.

두 구성 집합간에 projection이라는 두개의 함수가 존재한다. 하스켈에서 이 두 함수는 fst, snd라 부른다. 이들은 쌍에서 첫번쨰와 두번째 요소를 집어낸다.

```haskell
fst :: (a, b) -> a
fst (x, y) = x

snd :: (a, b) -> b
snd (x, y) = y
```

여기서 함수들은 그들의 아큐먼트를 패턴매칭하는것으로 정의되어있다. 패턴은 어떤 쌍 (x, y)에 대하여 x 혹은 y를 골라내는 패턴이다. 와일드 카드 패턴을 이용하여 더 단순하게 나타낼 수 있다.

```haskell
fst (x, _) = x
snd (_, y) = y
```

집합의 카테고리적 관점으로 시선을 옮겨 대상과 사상의 패턴을 정의해보자, 이것을 통하여 우리는 두 집합 a, b의 곱의 확장(construction)으로 안내할것입니다. 이 패턴은 대상 c에 대하여 다음과 같은 두개의 사상 p와 q를 지닌다.

```haskell
p :: c -> a
q :: c -> b
```

product에 해당하는 다음 패턴을 만족하는 c는 매우 많을것이다. Int와 Bool을 이용하여 몇가지 상황을 생각해보자. Int는 Int, Bool의 곱의 하나로 간주될수 있습니까? 아래와 같이 가능합니다.

```haskell
p :: Int -> Int
p x = x

q :: Int -> Bool
q _ = True
```

다음은 (Int, Int, Bool)에 대하여 생각해보자. 패턴매칭을 만족시키는 다음 두가지 사상이 가능할것이다.

```haskell
p :: (Int, Int, Bool) -> Int
p (x, _, _) = x

q :: (Int, Int, Bool) -> Bool
q (_, _, b) = b
```

Int를 이용했던 첫번쨰 후보군은 정보가 너무 적다(Int dimension 만 커버가 가능하다), 반대로 두번쨰 (Int, Int, Bool)을 이용했던 후보군은 정보가 너무 많다

Universal Construction을 위하여 랭킹을 적용해보자. 두가지 패턴의 인스턴스에 대하여 비교하여야한다. 대상 c와 이것의 두 projection p, q 그리고 다른 대상 c'와 이것의 두 projection p', q'를 비교하고자 한다. 만일 c' -> c인 사상 m이 있을때 c' <= c인 관계가 있다 하려면 어떻게 해야할까? 또한 c의 projection들이 c'의 projection들보다 더 낫다 혹은 더 일반적이다 말하려면 어떻게 하여야 할까? 프로젝션 p', q'가 m을 이용하여 p, q에서 재구성 될수있다는 것은 무슨의미일까?

```haskell
p' = p . m
q' = q . m
```

이 방정식을 보는 다른 관점은 m이 p', q'를 factorizes 한다는것이다. 자연수를 예로 들어보면 합성 .은 곱셈에 해당하고 m은 공약수(common factor)에 해당한다.

좀더 직관적인 이해를 위해서 (Int, Bool) 의 fst, snd projection을 이용하여 (Int, Bool)이 위에서 후보로든 두 경우(Int, (Int, Int, Bool)) 보다 더 낫다는 것을 보여주겠다.

![](https://bartoszmilewski.files.wordpress.com/2015/01/not-a-product.jpg)

```haskell
-- for Int
m :: Int -> (Int, Bool)
m = x = (x, True)

-- m과 fst의 합성을 통하여 p를 m과 snd의 합성을 통하여 q를 만들어낼수있다 -> (Int, Bool)이 더 낫다
p x = fst (m x) = x
q x = snd (m x) = True

-- for (Int, Int, Bool)
m (x, _, b) = (x, b)
```

위를 통하여 (Int, Bool)이 두가지 후보군보다 더 낫다 알 수 있다. 반대의 경우는 왜 성립하지 않는지 한번 보자. 만일 m의 역  m'이 있다면 우리는 p와 q를 이용하여 각각 fst, snd를 다시 만들어낼 수 있을것이다.

```haskell
fst = p . m'
snd = q . m'
```

첫번째 후보군의 예(Int)에서 q는 언제나 True를 반환했다.(이는 False를 포함하지 않는다) 고로 q를 이용해 snd를 만들어 낼 수 없다.

두번쨰 후보군 (Int, Int, Bool)에 대하여 우리는 p, q를 얻기위해 충분한 정보가 있다. 하지만 반대로 p를 이용하여 fst을 구해야할경우 트리플의 두번째 Int값은 무엇이든 될 수 있다.

위의 모든것을 합쳐 어떤 타입 c와 이의 두 프로젝션 p, q가 있을때 그것들을 factorize하는 c -> Cartesian produt(a, b)로 향하는 고유한 m이 있다 할 수 있다

```haskell
m :: c -> (a, b)
m x = (p x, q x)
```

이제 집합에서 벗어나 카테고리에서 두 대상간의 곱을 univresal construction을 이용하여 정의해보자. 위와같은 곱은 항상 존재하지 않을것이다. 하지만 존재한다면 그것은 고유한 isomorphism이라는 관점에서 고유할것이다.

> A product of two objects 𝑎 and 𝑏 is the object 𝑐 equipped with two projections such that for any other object 𝑐′ equipped with two projections there is a unique morphism 𝑚 from 𝑐′ to 𝑐 that factorizes those projections.

분해하는 함수 m을 만들어내는 고차함수를 종종 factorizer라 부르고 다음과같이 나타낼 수 있다

```haskell
factorizer :: (c -> a) (c -> b) -> (c -> (a, b))
factorizer p, q = \x -> (p x, q x)
```



### 5.6 Coproduct

![](https://bartoszmilewski.files.wordpress.com/2014/12/coproductpattern.jpg?w=150&h=94)

product의 쌍대로 coproduct가 있습니다. 곱 패턴에서 화살표를 뒤집으면 됩니다. 그러면 이번에는 a, b 에서 c로 향하는 두개의 인젝션(injection) 사상 i, j가 있게 됩니다.

```haskell
i :: a -> c
j :: b -> c
```

랭킹도 뒤집으로면 됩니다. i, j 인젝션이 있는 대상 c가 i', j' 인젝션을 지니는 대상 c'보다 낫다는 것은 c에서 c' 인젝션을 분해하는 m이 존재합니다.(object 𝑐 is “better” than object 𝑐′ that is equipped with the injections 𝑖′ and 𝑗′ if there is a morphism 𝑚 from 𝑐 to 𝑐′ that factorizes the injections:)

```haskell
i' = m . i
j' = m . j
```

 다음과 같은 상황에서 모든 패턴을 만족시키는 유일한 사상은 coproduct라 불립니다. 그리고 그것이 존재한다면 그것은 고유한 isomorphism에 따라 고유합니다.

> A coproduct of two objects 𝑎 and 𝑏 is the object 𝑐 equipped with two injections such that for any other object 𝑐′ equipped with two injections there is a unique morphism 𝑚 from 𝑐 to 𝑐′ that factorizes those injections.

집합의 카테고리에서 corproduct는 두 집합간의 분리합집합, 서로소 합집합(disjoint union)을 뜻합니다. 집합 a, b의 disjoint union의 원소는 집합 a나 b중 하나의 원소에 해당합니다. 만일 두 집합이 중첩되는 경우 분리합집합은 두개의 공통 부분 복사본을 지니게 됩니다(뭔말이야..) 부분합집합은 원소들이 그것들의 기원이 태그된 경우를 생각해보면 됩니다.

책에서는 c++ 기준으로 집합의 원소에 태그를 붙여 coproduct의 예를 나태냈는데 swift에서는 다음과 같이 나타낼 수 있을것같습니다.

```swift
enum Contact {
    case phoneNumber(_ value: Int)
    case email(_ address: String)
}

let phoneNumberContact: Contact = .phoneNumber(123)
let emailContact: Contact = .email("email_address")
```

하스켈에서 데이터 타입을 tagged union으로 나타내려면 데이터 생성자를 |로 분리해서 합성할 수 있습니다.

```haskell
data Contact = PhoneNum Int | EmailAddr String

helpdesk :: Contact
helpdesk = PhoneNum 22222
```

하스켈에서 product는 기본타입 pair로 구현되어있는것에 반하여 coproduct는 Either로 스탠다드 라이브러리 Prelude에 다음과 같이 정의되어 있습니다.

```haskell
data Either a b = Left a | Right b
```

Either는 a, b 두개의 데이터 타입과 각각의 생성자 Left, Right를 인자로합니다.

위에서 product의 factorizer를 정의하였듯이 우리는 coproduct도 할 수 있습니다.

```haskell
factorizer :: (a -> c) (b -> c) -> Either a b -> c
factorizer i j (Left a) = i a
factorizer i j (Right b) = j b
```

### 5.7 Asymmetry

우리는 두가지의 쌍대 정의에 대하여 알아보았다: Terminal & Initial Object / Product & Coproduct. 두가지 모두 하나의 정의를 뒤집어서 얻어냈지만 집합의 카테고리에서 Terminal Object는 Initial Objec와 Product는 Coproduct와 매우 다릅니다. 추후에 product는  곱셈처럼 작용하고 Terminal Object는 1처럼 작용하는것을 볼것이고 반대로 Coproduct는 덧셈, Iniital Object는 0 처럼 작동하는것을 볼것입니다.

이것은 집합의 범주에서 각각의 개념이 쌍대의 개념들과 대칭이 아님을 보여줍니다.

예를들어 엠티셋은 모든 셋으로 향하는 고유한 사상이 존재하지만 다시 돌아오는 사상은 없습니다. 싱글톤 셋에 대하여 어떤 셋에서든 들어오는 방향의 고유한 사상이 있고 또한 싱글톤 셋에서 엠티셋을 제외하고 모든 방향으로 나가는 사상도 존재합니다. 전에 보았듯이 터미널 오브젝트에서 다른 대상으로 나가는 사상은 대상의 원소를 골라내는 역할을 합니다.(엠티셋은 사상이 없어서 골라낼게 없습니다.)

(싱글톤 셋에 대한 product, coproduct 생략..)

이것은 집합 고유의 특성이 아니고 함수에 대한 성질이다. 함수들은 보통 비대칭적이다.

함수는 정의역의 모든 원소에 대하여 정의되어야한다. 하지만 공역의 모든 원소를 커버할필요는 없다. 함수는 공역의 원소중 하나를 선택한다. 만일 정의역의 범위가 공역보다 월등히 좁은 경우에 오르는 종종 함수가 정의역을 공역에 임베딩 시킨다 생각할것이다. 예를들어 우리는 싱글톤셋이 정의역인 함수는 그것의 엘리먼트를(()) 공역에 임베딩 시킨다 생각할 수 있다. 저자는 이를 임베딩이라 부르지만 다음과 같이 말한다합니다: 공역을 꽉채우는 함수를 subjective 혹은 onto라 부릅니다.

비대칭의 다른 원인은 정의역의 많은 요소가 공역의 공통의 요소로 매핑될수있다는 점 입니다. 극단적인 상황으로 모든 집합의 원소가 싱글톤셋으로 매핑되는것을 생각할 수 있습니다. 수학자들은 이를 injective 혹은 one-to-one이라 부릅니다.

물론 대칭적인 함수(bijections)도 존재합니다. 그리고 뒤집을수도 있습니다. 집합의 카테고리에서 isomorphism은 bijection과 동일합니다.