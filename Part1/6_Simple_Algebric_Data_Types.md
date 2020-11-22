## 6. Simple Algebric Data Types

product, coproduct라는 타입을 결합하는 두가지 방법은 알아보았다. 우리가 일상생활에서 만들어내는 수많은 데이터 타입들은 모두 이 2가지 방법을 이용하여 만들어낼 수 있다. 이점은 실질적으로 매우 중요하다. 데이터 구조의 많은 속성들은 합성가능하다. 예를들어 기본타입 데이터에 동등함을 위하여 비교하는 방법을 안다면 이 비교를 product와 coproduct로 일반화하는 방법을 알 수 있을것이다. 하스켈에서 string 혹은 더 큰 규모의 여러 서브셋들이 결합된 타입에 대하여 동등함, 비교, 변환등을 자동으로 이끌어낼 수 있다.

프로그래밍에서 product와 sum타입이 어떻게 나타나는지 알아보자.

### 6.1 Product Types

전통적으로 프로그래밍에서 두 타입의 product는 pair로 나타낸다.(하스켈에서 pair는 기본형) 

페어에서 순서는 중요하다. 예를들어 동일한 정보를 담고있음에도 불구하고 (Int, Bool)은 (Bool, Int) 형식의 페어로 대체될 수 없다. 하지만 둘은 isomorphism 적인 측면에서 대체가 가능하다. 다음과 같은 swap 함수에 의하여 동형을 이룰 수 있다.

```haskell
swap :: (a, b) -> (b, a)
swap (x, y) = (y, x)
```

단순하게 두 페어는 동일한 데이터를 다른 포맷으로 담고있다 생각하면 된다.

또한 우리는 여러 타입의 정보를 nested한 페어로 담을 수 있다. (nested한 페어는 튜플로 나타낼 수 있다.) 동일한 정보를 다른 방식으로 nested pair로 구성하는것은 동형이다. 

```haskell
-- nested pairs for a, b, c
((a, b), c)
(a, (b, c))

-- 두개의 타입은 다르다. 한 타입에서 다른 타입으로의 매핑하는 함수를 정의해보자
alpha :: ((a, b), c) -> (a, (b, c))
alpha ((x, y), z) = (x, (y, z))
-- alpha의 역함수
alpha_inv :: (a, (b, c)) -> ((a, b), c)
alpha_inv (x, (y, z)) = ((x, y), z)
```

마찬가지로 이경우에도 동형이다. 단지 동일한 데이터를 다른방식으로 담는것이라 생각할 수 있다.

Product type을 만드는것을 타입에 대한 binary operation으로 해석할수도 있다. 이와같은 관점에서 위의 동형은 모노이드에서 보았던 결합법칙과 매우 유사하다

```haskell
(a * b) * c = a * (b * c)
-- 모노이드의 경우 product를 구성하는 방법이 동일하지만 이번에는 동형이 동일하다
```

엄격하게 동일함을 판단하는것말고 동형만을 고려해서 좀더 생각해보자, 특히 유닛 타입에 대하여. 유닛 타입에서의 product는 곱셈에서 1과 같다. 실제로 어떤 타입을 유닛과 함께 페어로 만든다는것은 어떤 정보가 추가되지는 않는다.

```haskell
(a, ())
-- a에 대하여 동형이다
rho :: (a, ()) -> a
rho (x, ()) = x

rho_inv :: a -> (a, ())
rho_inv x = (x, ())
```

위를 정형화 시키면 Set(set의 카테고리) 는 monoidal category 이다. 이것은 카테고리이며 동시에 모노이드이다. 모노이드 범주의 정의 등은 나중에 거론하기로하자.

하스켈에서 product를 정의하는 방법이 더 있다.(이후에 보게 되겠지만 sum type과 함께 합성될때) 페어는 다음과같이 다시 정의할수 있다.

```haskell
data Pair a b = P a b
-- Pair, a, b는 서로 다른 데이터 타입을 나타낸다(Pair는 타입 생성자)
-- P는 생성자 이름이다.(P는 데이터 생성자)
-- Pair 타입의 생성자에 다른 두 타입을 전달하여 pair를 정의하였다.
-- data Pair a b = Pair a b 처럼 동일한 이름을 사용하여도 상관없다.

stmt :: Pair String Bool
stmt = P "This is statement is" False
-- 내장된 pair 선언을 위에서 변형했다고 생각할수도 있기때문에 아래와같이 써도 무방
stmt = (,) "This statement is" False

-- (,,) 이런형식으로 트리플을 만드는 것도 가능
-- 제네릭한 타입 말고 콘크리트한 product type을 정의 가능
data Stmt = Stmt String Bool
```

튜플이나 생성자의 인자가 많은 방식을 이용하여 프로그래밍을 하는것은 난잡하다.(어떤 컴포넌트가 무엇을 나타내는지 지속적으로 확인해야한다.) 컴포넌트에 이름을 부여하는것도 선호된다. 이를 하스켈에서는 record라부른다(C에서는 struct)

### 6.2 Records

간단한 예를 보자, 어떤 화학적 원소를 이름(String), 심볼(String), 원소번호(Int)로 합쳐 하나의 데이터 구조로 나타낸다 해보자. 단순하게 (String, String, Int) 트리플로 나타낼 수 있다. 패턴매칭을 이용해 각 구성요소를 추출해낼수있다.

```haskell
-- Helium의 심볼이 He이여하듯이 심볼이 원소 이름의 prefix인지 검사하는 함수
startsWithSymbol :: (String, String, Int) -> Bool
satrtswithSymbol (name, symbol, _) = isPrefixOf symbol name
```

위의 코드는 에러에 취약하고 유지보수도 힘들다. 아래와같이 record를 이용하는게 더 좋다.

```haskell
data Element = Element { name :: String
												, symbol :: String
												, atomicNumber :: Int }
```

변환 함수를 이용하여 트리플을 이용한 경우와 record를 이용한 경우가 동형임을 나타낼 수 있다.

```haskell
tuppleToElem :: (String, String, Int) -> Element
tuppleToElem (n, s, a) = Element{ name = n
																, symbol = s
																, atomicNumber = a }
																
elemToTuple :: Element -> (String, String, Int)
elemtoTuple e = (name e, symbol e, atomicNumber e)
-- record에서 필드이름은 필드값에 접근하기 위한 함수와 동일하다.
```

record를 이용하면 위에서 정의한 startsWithSymbol 함수를 더 읽기좋게 바꿀 수 있다

```haskell
startsWithSymbol :: Element -> Bool
startsWithSymbol e -> isPrefixOf (symbol e) (name e)
-- backquotes를 이용하여 isPrefixOf 함수를 인픽스로 오퍼레이터로 바꿀수있다.
-- 인픽스 오퍼레이터는 펑션콜보다 낮은 우선순위를 지니기때문에 이경우에 괄호를 생략 가능
startsWithSymbol e = symbol e `isPrefixOf` name e
```



### 6.3 Sum Types

카테고리에서 집합의 product가 product 타입이 되었듯이 coproduct는 sum type을 발생시킵니다. 하스켈에서 전통적인 sum type은 Either 입니다.

```haskell
data Either a b = Left a | Right b
```

Pair에서 보앗듯이 Either는 교환 가능하고(동형에 따라) nested할 수 있으며 nesting 순서는 상관없습니다(동형에 따라). 그래서 우리는 다음과 같이 트리플도 정의할 수 있습니다.

```haskell
data OneOfThree a b c = Sinistral a | Medial b | Dextral c
```

Set은 coproduct 적인 측면에서 (symmetric) monoidal category라 할 수 있습니다. 대응되는 바이너리 오퍼레이터는 disjoint sum입니다. 그리고 유닛 엘리먼트가 하는 역할은 이니셜 오브젝트가 수행합니다. 타입에 대하여 우리는 monoidal operator로서 Either와 Void가 있습니다. Either는 플러스로 Void는 0으로 생각할 수 있습니다. 실제로 sum type에 Void를 더하는것은 아무 의미가 없습니다.

```haskell
Either a Void
```

Right를 만들 수 없기 때문에 이는 a에 대하여 동형입니다. 오직 Left로만 Either a Void를 만들수 있습니다. 이것은 단순히  a를 캡슐화 하기만 합니다.

존재하지 않거나 존재하는 값에 대한 간단한 합성 타입은 c++에서 약간 트릭으로 사용됩니다(""이나 -1, null pointer와 같이) 이런 옵셔널한 타입을 하스켈에서는 Maybe type으로 나타냅니다

```haskell
data Maybe a = Nothing | Just a
```

Maybe는 두 타입의 합성입니다. 각각의 생성자에 해당하는 타입은 다음과 같습니다.

```haskell
-- Nothing은 nothing이라는 하나의 값만있는 열거형입니다. 다시말해 이는 singleton이고 unit type과 동일
data NothingType = Nothing
-- Just는 단지 타입 a를 감싸는 역할만 합니다.
data JustType a = Just a

-- Maybe를 다음과같이 인코딩할 수 있습니다.
data Maybe a = Either () a
```

C++에서 더 복잡한 sum type은 종종 가짜 포인터를 이용합니다. 포인터는 null이거나 특정 값을 가르킬수있습니다. List를 예를들어 봅시다.

```haskell
data List a = Nil | Cons a (List a)
```

하스켈과 C++의 타입에 제일 큰 점은 하스켈의 데이터 구조체는 불변이라는 점이다. 특정한 생성자를 이용하여 오브젝트를 만들었다면 오브젝트는 영원히 어떤 생성자가 이용되었고 어떤 인자가 주입되었는지 기억할것이다. 그러기 떄문에 Maybe 객체가 Just 생성자를 이용해 만들어졌다면 Nothing으로 절대 변하지 않는다. 마찬가지로 빈 리스트는 영원히 빈 리스트이다.

이 불변성이 생성을 역으로 돌릴 수 있게해준다. 생성에 이용되었던 객체들은 언제나 다시 분리될 수 있다. 이 해체(deconstruction) 작업은 패턴매칭으로 이루어지고 생성자를 패턴으로 재사용한다. 생성자의 인자는(있는경우) 변수로 (혹은 다른 패턴으로) 대체됩니다.

List의 경우 두개의 생성자가 있으므로 해체시에도 두개의 생성자 패턴이 존재합니다. 다음은 List에 대한 maybeTail이라는 함수 예제입니다.

```haskell
maybeTail :: List a -> Maybe (List a)

-- Nil constructor 패턴을 이용하여 Nothing을 반한합니다.
maybeTail Nil = Nothing

-- Cons constructor 패턴을 이용.
-- Cons의 첫번째 인자는 와일트카드 패턴(여기서 뭐든 상관없기때문)
-- Cons의 두번째 인자는 t에 바인딩되고 Just t를 반환
maybeTail (Cons _ t) = Just t
```

어떤 리스트인지에따라 위의 두 패턴매칭에 따라 걸러질것입니다. 만일 Cons를 이용해서 만들었다면 생성시 넘겨줫던 두번째 인자가 불러질것입니다.

C++에서 더 정교한 sum type은 상속입니다. 공통 조상을 지니는 클래스 패밀리는 하나의 변형 유형으로 생각할 수 있습니다. C++에서 vtable pointer를 기반으로 어떤 펑션콜을 디스패치하는것은 하스켈에서 생성자 패턴을 이용해 찾는것으로 수행됩니다.



### 6.4 Algebra of Types

각자 별도로 생각해서 product와 sum type은 매우 다양한 데이터 구조를 정의하는데 유용합니다. 하지만 진짜 이점은 두개를 합성했을때 발생합니다.

한것들을 요약해봅시다. 우리는 시스템의 기반이 되는 두개의 교환 가능한 monoidal 구조에 대하여 보았습니다. Void이 중립적인 원소(neutral element) 역할을 하는 sum type에 대하여 보았고,  unit이 중립원소 역할을 하는 product type도 보았습니다. 그리고 이것들은 덧셈, 곱셈과 유사하다고 생각하려합니다. 이 비유에서 Void는 0에 unit 은 1에 해당합니다.

이 비유를 어디까지 확장할수있을지 생각해봅시다. 예를들어 어떤값에 0을 곱하면 0이 됩니까? 다시말해서 product type의 한 원소가 Void라면 이는 Void와 동형입니까? (Int, Void)가 Void랑 동형입니까?

이 페어를 만들기 위해서는 두개의 값이 필요합니다. 하지만 Void는 만들 수 없습니다 고로 어느타임 a에 대해서든 (a, Void)는 불가능합니다. 고로 (a, Void)와 a는 동형입니다. 이것은 마치 a x 0 = 0인것과 비슷합니다.

곱셈과 덧셈에 대한 교환법칙을통해 생각해봅시다.

```haskell
a x (b + c) = a x b + a x c
```

여기에는 product도 있고 sum type도 있습니다. 다음과같이 나태내봅시다

```haskell
(a, Either b c) = Either (a, b) (a, c)
```

아래와 같이 변환하는 함수를 만들 수 있습니다.

```haskell
prodToSum :: (a, Either b c) -> Either (a, b) (a, c)
prodToSum (x, e) = 
	case e of
		Left y -> Left (x, y)
		Right z -> Right (x, z)
		
sumToProd :: Either (a, b) (a, c) -> (a, Either b c)
sumToProd e
	case e of
		Left (x, y) -> (x, Left y)
		Right (x, z) -> (x, Right z)
```

case 구문은 함수 내에서 패턴매칭을 하는데 쓰였습니다. 패턴이 일치할 경우에 평가할 표현식이 화살표 다음에 옵니다. 

```haskell
prod1 :: (Int, Either String Float)
prod1 = (2, Left "Hi")
-- prof1을 prodToSum에 넘기면 (2, " Hi")가 나옴
```

역관계에있는 두함수는 단순히 정보를 담는 방식을 다시 정렬하는 역할을 합니다.(같은 데이터를 다른 포맷으로 보관)

수학적으로 위와같이 밀접한 관계의 모노이드들을 semiring이라 부릅니다. 타입의 substraction을 정의할수는 없기때문에 완벽한 ring은 아닙니다.(semiring은 rig라 불립니다 -  ring without n(negative)). rig를 형성하는 자연수의 특성을 타입에 대한것으로 바꾸어 말할수있습니다.

| Numbers   | Types                          |
| --------- | ------------------------------ |
| 0         | Void                           |
| 1         | ( )                            |
| a + b     | Either a b = Left a \| Right b |
| a x b     | (a, b) or Pair a b = Pair a b  |
| 2 = 1 + 1 | data Bool = True \| False      |
| 1 + a     | data Maybe = Nothing \| Jsut a |

위의 리스트는 방정식의 해답을 정의하기때문에 매우 흥미롭습니다. 우리가 정의하고 있는 타입은 방정식의 양쪽에 나타납니다.

```haskell
data List a = Nil | Cons a (List a)
-- List a를 x로 대체해면 다음과 같은 방정식이 나옵니다
x = 1 + a * x
```

우리는 타입을 빼거나 나눌 수 없기때문에 일반적인 대수학으로는 위의 방정식을 풀수는 없습니다. 하지만 우변에 위치한 x를 계속하여 (1 + a * x)로 대체하고(재귀적이니?) 분배법칙을 적용하면 다음과같은 수열을 이끌 수 있습니다.

```haskell
x = 1 + a*x
x = 1 + a(1 + a*x) = 1 + a + a*a*x
x = 1 + a + a*a*(1 + a*x) = 1 + a + a*a + a*a*a*x
....
x = 1 + a + a*a + a*a*a + a*a*a*a + ....
```

product는(tupple) 무한대로 뻗어나갈것이고 다음과 같이 해석됩니다: 리스트는 비어있거나(1), 싱글톤이거나(a), 페어이거나, 트리플이거나 ..... 이것은 리스트가 무엇인지와 정확히 일치합니다.

리스트 말고 더 있습니다. 우리는 추후에 펑터(functor)와 부동점(fixed point)를 배운이후 다시 돌아와 다른 재귀적인 자료 구조에 대하여 더 알아볼 것 입니다.

기호변수로 방정식을 푸는것은 대수적입니다. 이런 유형에 대하여 대수적 데이터 유형(algebraic data type)이라는 이름을 부여합니다.

마지막으로 매우 중요한 대수적 데이터 유형에 대한 해석은 알아봅시다. a, b의 Product type은 a와 b타입에 해당하는 값 모두 무조건 들고있어야합니다. 반대로 sum type의 경우 a나 b 둘중 하나의 값만을 들고있어야합니다. 논리적으로 and와 or도 rig 입니다. 역시 다음과 같이 타입이론에서 나타낼 수 있습니다.

| Logic    | Types                          |
| -------- | ------------------------------ |
| false    | Void                           |
| true     | ( )                            |
| a \|\| b | Either a b = Left a \| RIght b |
| a && b   | (a, b)                         |

이 비유는 더 심오해집니다. 이것은 커리 하워드의 논리와 타입 이론간 isomorphism의 기초입니다. 함수 타입에 대하여 이야기하고 난 이후에 다시 이야기할것입니다.

