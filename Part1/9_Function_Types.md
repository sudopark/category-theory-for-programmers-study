## 9. Function Types

함수의 타입은 다른 타입과는 다릅니다.

정수형(Integer)를 예를들면 이들은 그냥 정수들의 집합입니다. Bool은 원소가 2개인 집합입니다. 하지만 다음과 같은 함수타입은 a -> b 더 많은 의미가 있습니다. 이는 대상 a, b간의 사상 집합을 의미합니다. 카테고리에서 대상간 사상의 집합은 hom-set으로 부릅니다. Set 카테고리에서 모든 hom-set은 그 자체로 동일한 카테고리 객체입니다.(결국 이는 집합이기 때문에). 다른 카테고리에서는 다른데 이경우 external hom-set이라 부릅니다.

함수 타입을 특별하게 만드는 이유는 Set 카테고리의 자체 참조특성(self-referential nature) 입니다. 그러나 일부 카테고리 에서는 hom-set을 나타내는 객체를 구성하는 방법이 있습니다. 이러한 객체를 internal hom-set이라 부릅니다.(뭔소리야..)



### 9.1 Universal Construction

함수의 타입이 집합이라는 것을 잊고 함수 타입 혹은 보다 일반적으로 internal hom-set을 처음부터 구성해 봅시다. 평소와 같이 Set 카테고리에서 시작하지만 집합의 속성을 사용하지 않도록 주의하여 구성이 다른 범주에 자동으로 적용되도록 합시다.

함수타입은 아규먼트와 리턴타입의 관계를 나타내기 때문에 타입이 합성되어있다 생각할 수 있습니다. 우리는 이미 복합 타입의 구성(객체간의 관계를 포함하는)을 보았습니다. product와 coproduct를 정의하기 위해 universal construction을 이용하였습니다. 이번에도 함수 타입을 정의하기 위해 같은 방법을 이용할 것 입니다. 이를 위하여 패턴이 필요하고 이는 3가지의 대상으로 구성됩니다.(만드려고하는 함수 타입, 인풋 타입, 리턴 타입)

이 3가지 패턴을 연결하는 패턴을 **function application** 혹은 **evaluation**이라 부릅니다. 함수 타입의 후보자를 z, 입력을 a, 리턴을 b라 부릅시다. 3가지 타입 중 두가지 타입(입력, 출력)은 고정입니다.

우리는 또한 application을 지니고 있고 이것은 매핑입니다. -> 이 매핑을 어떻게 패턴에 통합 하여야 할까요? 우리가 대상 내부를 볼 수 있다면 다음과 같을 것 입니다. 함수 f(z의 원소)와 x(a의 원소)를 이용하여 pair를 만듭니다. 그리고 이것을 f x(x에 f를 적용 -> b의 원소) 로 매핑합니다.

![](https://bartoszmilewski.files.wordpress.com/2015/03/functionset.jpg?w=300&h=263)

각각의 (f, x) 쌍을 다루기보다는 타입 z와 a의 product에 대하아여 이야기 할 것 입니다. 곱 z x a 은 하나의 객체입니다. 여기서 b로 가는 화살표를 g라 합시다.

이제 패턴이 완성 되었습니다: 두 대상의 곱이(z, a) 사상 g를 이용하여 b와 연결

일단 곱이 정의되지 않거나 모든 대상의 페어에 곱이 정의되지 않아도 위의 패턴을 정의할 수 있는가에 대하여 의문을 가질 수 있습니다.답은 그럴수 없다 입니다. Product type이 없다면 함수 타입도 존재하지 않습니다(추후에 exponetial을 이야기할때 이부분에대해 다시 언급할 예정입니다.)

universal construction을 위하여 패턴은 정의했고 가능한 후보자들 중에 랭킹을 적용해 봅시다. 

![](https://bartoszmilewski.files.wordpress.com/2015/03/functionranking.jpg?w=300&h=241)

위의 그림에서  z와 z x a에서 b로 가는 사상 g가 다른 후보  z'와 g'보다 더 낫다고 합시다. 그리고 z'에서 z로의 고유한 매핑 h가 있는 경우에만 application g를 application g'의 요소로 분리합니다. 자세히 보면 h가 z' x a 를  z x a로 매핑하고 있다 볼 수 있습니다. 곱 그 자체로 펑터입니다. 그렇기 때문에 곱은 사상 쌍도 리프팅 할 수 있습니다.

곱에서 두번째 요소 a는 건들이지 않기때문에 (h, id)로 사상을 리프팅 할 것 입니다.(id는 a의 항등사상)

이로서 하나의 application g를 다른 application  g'에서 분리할 수 있습니다.

```haskell
g' = g . (h x id)
```

universal construction의 마지막 단계로 보편적으로 최상의 대상을 골라야 하고. 이를 a => b라 부릅시다. 이 대상은 eval이라 불리는 자체 application과 함께 제공됩니다. - (a => b) x a에서 b로 가는 사상. 만일 다른 함수 객체 후보자들이 이것으로 고유하게 매핑된다면 a=>b는 최상의 후보자 입니다. 그러면 application 사상 g는 eval을 통해 분리됩니다. 이는 앞서 정의한 랭킹중에 최상의 결과입니다.

![](https://bartoszmilewski.files.wordpress.com/2015/03/universalfunctionobject.jpg?w=300&h=231)

```haskell
-- Formally
-- a에서 b로 함수객체는 사상과 함께 a=>b의 객체입니다.
eval :: ((a=>b) x a) -> b
-- z의 모든 객체에 대한 사상
g :: z x a -> b
-- 다음의 고유한 사상이 존재
h :: z -> (a => b)
-- eval을 통한 분해
g = eval . (h x id)
```

카테고리에서 a=>b 객체가 존재한다 보장할수는 없지만 Set에는 항상 존재합니다. 더욱이 이 객체는 hom-set Set(a, b)와 동형 입니다. 



### 9.2 Currying

이번에는 사상 g를 z, a 두 변수에 대한 함수로 생각해 봅시다. 곱에서 시작하는 사상은 두 변수를 받는 함수와 비슷합니다.

```haskell
g :: z x a -> b
```

반면에 위에서 본 보편적인 성질에 따라 모든 g에 대하여 z -> (a=>b)로 가는 고유 사상 h가 있음을 알 수 있습니다.

```haskell
h :: z -> (a => b)
```

Set에서 h는 z타입 인자를 하나 받아  a에서 b로 가는 함수를 반환하는 함수를 뜻합니다. h는 이렇기 때문에 고차함수입니다. 그러므로 universal construction은 두개의 인자를 받는 함수와 하나의 인자를 받아 함수를 반환하는 함수간에 1:1 대응 관계가 있음을 나타냅니다. 이 대응 관계를 **currying**이라 부르고 여기서 h를 g의 curried version이라 부릅니다.

어떤 g이든 고유한 h가 존재하기 때문에 이 대응관계는 1:1 입니다. 그리고 h가 주어지면 아래의 공식을 이용하여 두개의 인자를 받는 함수 g를 다시 만들 수 있습니다. 그리고 g는 h의 uncurried version이라 불립니다.

```haskell
g = eval . (h x id)
```

커링은 하스켈의 구문에 내장되어 있습니다. (함수를 반환하는 함수)

```haskell
a -> (b -> c)
-- 두개의 인자를 받는 함수: a -> b -> c

catstr :: String -> String -> String 
catstr s s' = s ++ s'

catstr' :: String -> String -> String
catstr' s = \s' -> s ++ s'

-- 하나의 인수에만 부분적으로 적용되어 다음과 같이 단일 인수 함수를 생성 가능
greet :: String -> String
greet = catstr "Hello"
-- catstr "Hello" 는 String -> String 함수를 반환, greet 타입 시그니처와 일치
```

엄격히 말해서 두개의 인자를 받는 함수는 하나의 쌍(product type)을 받는것 입니다.

```haskell
(a, b) -> c

-- 두 표현 사이를 변경하는것은 당연히 curry, uncurry라 불린다.
curry: ((a, b) -> c) -> (a, b, c)
curry f a b = f (a, b)
uncurry :: (a -> b -> c) -> ((a, b) -> c)
uncurry f (a, b) = f a b

-- curry는 함수 객체의 universal construction을 위한 factorizer
factorizer :: ((a, b) -> c) -> (a -> (b -> c))
factorizer g = \a -> (\b -> g(a, b))
```



### 9.3 Exponentials

수학적으로 함수 객체나 두 객체간의 internal hom 객체는 **exponential**이라 부르고 𝑏^a로 나타냅니다. 이 표시법이 이상해 보일수 있습니다. 하지만 함수와 product간의 관계를 생각해보면 확실히 이해할 수 있습니다.

유한범위의 타입간의 함수를 생각해보세요. 유한범위의 타입에는 유한범위의 값이 있습니다. (Bool, Char, Int, Double..) 원칙적으로 이와같은 함수들은 meorized 되거나 룩업테이블에서 조회할 수 있는 구조로 바꿀 수 있습니다. 이는 함수(사상) 그리고 함수 타입(객체) 간 동등함을 나타냅니다.

예를들어 (순수 함수의 경우에) Bool을 받는 함수는 입력이 False이거나 True인 두가지 경우에 따른 가능한 결과값이 있을것입니다. Int의 경우에는 Int의 사이즈에 해당하는 만큼일것이고요. 

함수와 데이터 타입 사이에는 동등성이 있습니다. 무한 범위의 타입에 대하여 이를 매칭하기 위해서 무한한 메모리가 필요합니다. 하지만 하스켈은 lazy한 언어 이기 때문에 레이지하게 평가된 (무한) 데이터 구조와 함수 사이의 경계가 모호합니다. 이 함수 vs 데이터 쌍대성은 범주 형 지수 객체를 사용하여 하스켈의 함수 타입을 식별하는 방법을 설명합니다. 그리고 이는 우리가 데이터에 대하여 생각하는 방식과 더 비슷합니다.



### 9.4 Cartesian Closed Categories

지금까지 집합의 카테고리로 예를 들어 왔지만 더 큰 개념의 카테고리가 있습니다. 이를 Cartesian closed라 부르고 Set은 이런 카테고리의 한 예 입니다. Cartesian closed 카테고리는 다음을 지녀야 합니다.

1. Terminal object
2. A product of any pair of objects, and
3. An exponential for any pair of objects.

exponential을 반복되는 곱이라 생각한다면 Cartesian closed 카테고리는 임의의 인자(arity) 하나를 지원하는 곱들이라 볼 수 있습니다?(hen you can think of a Cartesian closed category as one supporting products of an arbitrary arity. ) 특히 터미널 오브젝트는 제로 오브젝트의 곱 혹은 특정 오브젝트의 0제곱 이라 볼 수 있습니다.

컴퓨터 사이언스에서 이 카테고리가 흥미로운 점은 단순한 람다식 모델을 제공한다는 점 입니다. 그리고 이들은 모든 유형의 프로그래밍 언어의 기초를 이룹니다.

터미널 오브젝트와 곱은 쌍이 있습니다. - 이니셜 오브젝트와 쌍대곱. 그리고 역시 Cartesian closed category는 이들도 지원합니다. 그리고 쌍대곱에 대하여 곱이 분배법칙을 이룰 수 있는 경우를 bicartesian closed category라 부릅니다.

```haskell
a x (b + c) = a x b + a x c
(b + c) x a = b x a + c x a
```



### 9.5 Exponentials and Algebric Data Types

함수타입을 지수타입으로 해석하는것은 대수적 데이터 체계에 매우 적절합니다.

#### 9.5.1 Zeroth Power

```haskell
a^0 = 1
```

카테고리에서 0은 이니셜 오브젝트 이고 1은 파이널 오브젝트 입니다. 그리고 동등은 동형(isomorphism)을 뜻합니다. 익스포넨셜은 인터널 홈셋 객체 입니다. 이 특정 지수는 초기객체(0)에서 임의의 객체 (a)로 이동하는 사상의 집합을 나타냅니다. 초기객체의 정의에 따라 이와 같은 사상은 오직 한개 입니다. 고로 홈셋 C(0, a)는 싱글톤 셋 입니다. 집합의 카테고리에서 싱글톤셋은 터미널 오브젝트 입니다.

하스켈에서 위를 나타내려면 0을 Void와 1을 Unit type ( )으로 그리고 지수형태는 함수로 대체하면 됩니다. 일전에 Void -> a로 사상은 유일합니다(일전에 이를 absurd라는 함수로 불렀었습니다.)

#### 9.5.2 Powers of One

```haskell
1^a = 1
```

위는 집합의 카테고리에서 해석될때 터미널 오브젝트의 정의를 다시 나타냅니다. -> 어떤 대상에서 터미널 대상까지의 사상은 유일함을 뜻합니다. 일반적으로 이 사상의 internal hom-set은 터미널 객체와 동형입니다.

하스켈에서 어떤 타입이든 받아 유닛을 반환하는 함수를 보았고 유일하다는것 또한 보았습니다. - 이 함수는 unit이라 부릅니다. 또한 이를 어떤 타입과 ( )를 const 함수에 적용한것으로 해석할 수 도 있습니다.

#### 9.5.3 First Power

```haskell
a^1 = a
```

이것은 관찰(observabation)을 뜻합니다. 터미널 오브젝트에서 어떤 대상으로의 사상은 그 대상의 원소를 집어낸다를 뜻합니다. 이런 사상들의 집합은 대상 자체와 동형입니다. 하스켈에서 타입 a의 원소와 그것을 골라내는 함수 ( ) -> a는 동형입니다.

#### 9.5.4 Exponentials of Sums

```haskell
a^(b+c) = a^b x a^c
```

카테고리적으로 위는 두 대상의 coproduct의 exponetial은 두 exponential의 product과 동형임을 뜻합니다. 하스켈에서 이 대수적 정체성은 매우 실용적입니다. 이는 두 타입의 합을 인자로 받는 함수는 각각의 타입을 인자로 하는 함수의 곱과 동등함을 뜻합니다. 이것은 합에 대한 함수를 정의할때 사용됩니다. 하나의 함수 정의를 case 문과 함께 쓰는것 대신에 분리하여 각각의 생성자를 분리하여 함수를 정의하기도 합니다.

```haskell
f :: Either Int Double -> String
f (Left n) = if n < 0 then "Negative int" else "Positive int"
f (Right x) = if x < 0.0 then "Negative double" else "Positive double"
```

#### 9.5.5 Exponentials of Exponentials 

```haskell
(a^b)^c = a^(bxc)
```

위는 커링을 지수형식으로 나타낸 경우입니다. 함수를 반환하는 함수는 곱을 인자로하는 함수(인자가 두개)로 하는 함수와 같음을 뜻합니다.

#### 9.5.6 Exponentials over Products

```haskell
(a x b)^c = a^c x b^c
```

하스켈에서 pair를 반환하는 함수는 함수들의 페어와 같습니다.

이러한 단순한 고등학교 대수적 정체성을 카테고리 이론으로 끌어올려 함수형 프로그래밍에 실용적으로 적용하는 방법은 놀랍습니다.

### 9.6 Curry-Howard Isomorphism

일전에 로직과 대수적 기본 데이터 구조간에 일치함이 있음을 다뤘었습니다. Void와 ( )은 각각 false와 true에 대응합니다. Product와 Sum 타입은 각각 논리적 conjuction ∧ (AND), disjunction ∨ (OR)에 대응됩니다. 그리고 함수 타입은 앞서 정의한 =>의 논리적 구현과 대응됩니다. 다시말해서 a -> b의 타입은 "if a then b"로 읽어질을 수 있습니다. 

커리하워드의 동형에 따라 모든 유형은 명제로서 해석될 수 있습니다. - 진실 또는 거짓일수 있는 statement나 judgement로써. 이와같은 명제는 실존하는?(inhabited) 타입이면 참이고 그렇지 않으면 거짓입니다. 특별 해당하는 함수 타입이 inhabited하는 경우 논리적 의미는 참이며 해당하는 타입의 함수가 존재함을 의미합니다.(~~말 참 어렵게하네..~~) 함수 구현이 존재함은 정리를(theorems) 증명합니다.  그렇기에 **프로그래밍을 작성하는 것은 정리를 증명하는것과 같습니다.** 

```haskell
eval :: ((a -> b), a) -> b
-- 이는 다음의 사상을 하스켈에서 구현한것과 같다
-- 𝑒𝑣𝑎𝑙 ∷ (𝑎 ⇒ 𝑏) × 𝑎 → 𝑏
```

이를 커리하워드의 동형을 이용하여 이 시그니쳐를 논리적 술어?(logical predicate)로 바꾸어 봅시다

```
((𝑎 ⇒ 𝑏) ∧ 𝑎) ⇒ 𝑏
```

이 문장은 다음과 같이 읽을 수 있습니다. 만일 a 로부터 b가 나오는것이 참이라면 그리고 a도 참이라면 b도 무조건 참이다. 직관적으로 완벽하게 이해되며 이는 고대부터 modus ponens로 알려져 있습니다. 우리는 함수를 구현하여 이 정리를 증명할 수 있습니다.

```haskell
eval :: ((a -> b), a) -> b
eval (f, x) = f x
-- a를 만드는 함수와 a의 원소 쌍이 있으면 b타입의 원소를 만들어 낼 수 있음
```

일부러 거짓 명제를 드는것은 어떨까요? 만일 a나 b가 참이면 a는 무조건 참이다는 명제를 들어봅시다.

```
𝑎∨𝑏⇒𝑎
```

이것은 분명하게도 거짓입니다. (a가 false, b가 true 인 경우)

```haskell
Either a b -> a
-- Right가 불린경우에 대하여 이 함수를 구현할 수 없습니다. 
```

위에서 정의했던 absurd 함수를 다시 생각해 봅시다.

```haskell
absurd :: Void -> a
```

```
Void를 false로 바꾸어 번역하면 다음과 같음
false => a
```

모든것은 거짓에서 부터 나온다(ex falso quodlibet). 아래는 위 명제(함수)를 증명가능한 한가지 증명(구현)을 하스켈에서 나타낸 것 입니다.

```haskell
newtype Void = Void Void
absurd (Void a) = absurd a
```

위 정의는 값을 구성하기 위하여 값을 제공해야 하기 때문에 불가능 합니다. 그러므로 absurd 함수는 절대 불려질 수 없습니다.

위는 모두 재미있는 예 입니다. 커리하워드 동형이 실용적인 점이 있을까요? 아마 일반적인 프로그래밍에는 없을것 입니다. 하지만 Agda 또는 Coq와 같은 프로그래밍 언어는 커리하워드 동형을 활용하여 정리를 증명합니다.

컴퓨터는 단순히 수학자들의 작업을 돕는것이 아니라 수학의 기초를 혁신하고 있습니다. 이분야에서 제일 근래에 제일 핫한 주제는 Homotopy Type Theory 입니다.(타입 이론에서 파생 되었습니다.) 이것은 모두 불, 정수, 곱, 쌍대곱, 함수 타입 이런것들로 구성되어 있습니다. 그리고 이론은 Coq와 Agda에서 공식화되고 있습니다. 컴퓨터는 여러가지 방법으로 세상을 바꾸고 있는 중 입니다.


