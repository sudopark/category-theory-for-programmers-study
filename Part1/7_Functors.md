## 7. Functors

functor는 매우 간단하고 강력한 개념입니다. 카테고리 이론은 이와같은 간단하고 강력한 아이디어들로 가득차 있습니다. functor는 카테고리간 매핑입니다. 두 카테고리 C, D가 있을때 functor F는 C의 대상을 D의 대상으로 매핑합니다. 이것은 대상에 대한 함수입니다. C 카테고리의 대상 a가 있을때 이것이 카테고리 D로의 상을 Fa로 부를것입니다. 하지만 a는 단순한 대상만이 아닙니다. 대상은 물론 사상도 연결되어있습니다. 펑터는 사상도 매핑합니다. 이것은 사상에 대한 함수입니다. 그러나 펑터는 사상을 무의미하게 매핑하지 않습니다. - 연결을 유지시킵니다. 카테고리 C에 a -> b에 해당하는 함수 f가 있을때 이것이 D로의 상 Ff는 a의 상과 b의 상을 연결합니다.

```haskell
f :: a -> b
Ff :: Fa -> Fb
```

![](https://bartoszmilewski.files.wordpress.com/2015/01/functor.jpg)

위에서 보듯 펑터는 **카테고리의 구조를 보존시킵니다.** 본래 카테고리에서 있는 연결이 다른 카테고리에서도 존재함을 뜻합니다. 단순히 카테고리의 구조적 말고도 사상의 합성도 보존시킵니다. 사상 f, g의 합성이 h라 한다면 이것의 상은 다음과 같이 나타낼 수 있습니다.

```haskell
h = g . f
Fh = Fg . Ff
```

마지막으로 카테고리 C에 존재하는 모든 항등사상이 카테고리 D로 매핑되어야합니다.

```haskell
Fida = idFa
```

이 조건이 펑터를 다른 함수와 비교하여 더욱 한정적이게 합니다. 펑터는 카테고리의 구조를 유지시켜야 합니다. 만일 어떤 카테고리에 존재하는 수많은 대상과 이들의 사상이 얽혀있을때 펑터는 이것들을 한개라도 찢어서는 안됩니다. 이 관계는 대상들을 쪼개거나 여러 사상을 이어붙이는 것 일수도 있습니다. 하지만 절대로 이 관계를 떼어내서는 안됩니다. 이 찢어지지 않는 조건은 미적분에서 연속성의 조건과 유사합니다. 이런의미에서 펑터는 "연속적" 입니다.(펑터에 더 제한적인 연속성 개념이 존재하긴 합니다.) 함수와 마찬가지로 펑터는 중첩(injective?)되거나 임베딩(subjective?) 되기도 합니다. 소스 카테고리가 대상 카테고리보다 더 작을때 임베딩이 더 부각됩니다. 극단적으로 소스 카테고리는 singleton 카테고리 일수있습니다(대상과 항등사상만 존재하는). 싱글톤 카테고리에서 다른 모든 카테고리로의 펑터는 대상을 선택하는 역할을 합니다. 이것은 싱글톤셋이 하던것과 매우 유사합니다. 최대로 collapsing하는 펑터를 constant functor Δ𝑐 로 부릅니다. 이경우에 소스 카테고리에 위치하는 모든 대상은 대상카테고리의 한가지 대상으로 매핑합니다. 이것은 또한 소스카테고리에 존재하는 모든 사상을 idc로 매핑시킵니다. 이것은 마치 블랙홀 같습니다. 모든것을 단 하나로 압축시킵니다. 극한(limit)과 쌍대극한(colimit)을 이야기한뒤에 더 많은 펑터에 대하여 이야기 나눌 것 입니다.



### 7.1 Functors in Programming

현실로 돌아와서 프로그래밍에 대하여 생각해봅시다. 우리는 타입과 함수에 대한 카테고리가 있습니다. 우리는 펑터가 이 카테고리를 자신으로 매핑시키는 것이라 말할 수 있습니다. - 이와같은 펑터를 endofuntor라 부릅니다. 타입의 카테고리에서 endofunctor는 무엇입니까? 우선 이것은 타입에서 타입으로 매핑시킵니다. 아마 눈치를 채지는 못했겠지만 우리는 이미 이런 매핑을 아주 많아 봐왔습니다. 다른 유형에 대하여 매개변수화된 타입에 대하여 말하고 있습니다. 몇가지 예를 더 살펴보겠습니다.



#### 7.1.1 The Maybe Functor

Maybe의 정의는 타입 a를 Maybe a 타입으로 매핑하는 것입니다.

```haskell
data Maybe a = Nothing | Just a
```

여기서 디테일한 부분이 중요합니다. 메이비 자채는 타입이 아닙니다. 이것은 타입의 생성자입니다. Int, Bool과 같은 타입을 타입 아규먼트 a에 넣어줘야합니다. 그래야 타입으로 바뀝니다. 아무런 아규먼트가 없는 Maybe는 타입에 대한 함수를 나타냅니다. 하지만 Maybe를 펑터로 바꿀 수 있을까요?(지금부터 나올 프로그래밍에서의 펑터들은 모두 엔도펑터를 의미합니다.) 펑터는 대상만 매핑하는 것이 아니라(여기서는 타입) 사상도 매핑합니다(여기서는 함수). a -> b인 함수가 있을때 우리는 다음과 같이 우리는 펑터의 정의에 의해 Maybe a -> Maybe b인 함수도 만들어야합니다. Maybe의 두가지 생성자에 대하여 우리는 이 두경우를 고려해야합니다. Nothing이라면 Nothing을 반환하게 하면 됩니다. just라면 함수 f를 just의 아규먼트에 적용하면 됩니다.

```haskell
f' :: Maybe a -> Maybe b
f' Nothing = Nothing
f' Just x = Just (f x)
```

하스켈에서 우리는 fmap이라는 고차함수로 사상의 매핑을 구현합니다. Maybe는 다음과 같은 시그니처를 지닙니다.

```haskell
fmap :: (a -> b) -> (Maybe a -> Maybe b)
```

종종 fmap은 함수를 리프팅 한다 불립니다. 리프팅된 함수는 Maybe의 값에 대하여 동작합니다. 커링에따라 시그니쳐는 두가지 방식으로 해석될 수 있습니다. 아규먼트가 하나인 함수 (a -> b)를 (Maybe a -> Maybe b) 함수로 바꾼다 / 혹은 아규먼트가 두개인 함수가 Maybe b를 반환한다.

```haskell
fmap :: (a -> b) -> Maybe a -> Maybe b
```

Maybe에 대한 fmap의 구현은 다음과 같습니다

```haskell
fmap _ Nothing = Nothing
fmap f (Jsut x) = Just (f x)
```

타입 생성자 Maybe가 fmap과 함께 펑터를 형성한다는 것을 보여주기 위하여 우리는 fmap이 identity와 합성을 보존한다는 것을 보여야 합니다. 이를 "펑터 법칙"이라 부르지만 단순히 카테고리의 구조를 보존한다는것을 보장합니다.



#### 7.1.2 Equational Reasoning

펑터의 법칙을 증명하기위해 하스켈에서 매우 일반적인 증명 방식인 equational reasoning을 이용할겁니다. 하스켈 함수들은 등식 형식으로 되어있기 때문에 매우 장점을 가집니다. either의 인라이닝 펑션에 대해 생각해보십쇼, 반대로 식을 함수로 리팩토링 하는것을 생각해보세요. 항등함수를 이용하여 예를 들어 봅시다

```haskell
id x = x
```

x를 y로 이름을 바꾸거나 id (y + 2) = (y + 2) 이런식으로 바꿀 수 있습니다. 만일 함수가 패턴매칭으로 정의된경우 각각의 서브 정의를 독립적으로 사용할 수 있습니다. 예를들어 위의 fmap의 정의에서 fmap f Nothing을 Nothing으로 대채할 수 있습니다. 실제적으로 이것이 어떻게 동작하는지 알아봅시다,  identity가 어떻게 펑터에서 보존되는지 먼저 알아보죠

```haskell
fmap id = id
```

Nothing과 Just 두가지 케이스를 고려해야합니다. (좌변을 우변으로 변형하기 위해 하스켈의 수도코드를 이용할것입니다.)

```haskell
-- for Nothing
fmap id Nothing = Nothing
fmap id Nothing = id Nothing -- id의 정의에 의하여 Nothing을 id Nothing으로 대체(리팩토링)

-- for Just
fmap id (Just x) = Just (id x)
fmap id (Just x) = Just x -- id의 정의에 따라 id x를 x로 대체
fmap id (Just x) = id (Just x) -- 다시 id의 정의에 따라 Just x를 id (Just x)로 변환
```

다음으로 fmap이 합성을 보존하는지 알아봅시다

```haskell
-- fmap (g . f) = fmap g . fmap f

-- for Nothing
-- fmap _ Nothing = Nothing => fmap g Nothing = Nothing / fmap f Nothing = Nothing
fmap (g . f) Nothing = Nothing -- fmap의 정의에따라 우변 전체는 Nothing
fmap (g . f) Nothing = fmap g Nothing -- fmap의 정의에 따라 Nothing을 fmap g Nothing으로 변경
fmap (g . f) Nothing = fmap g (fmap f Nothing) -- fmap의 정의에 따라 이번엔 Nothing을 fmap f Nothing으로 변경

-- for Just
-- fmap f (Just x) = Just (f x)
fmap (g . f) (Just x) = Just ((g . f ) x)
fmap (g . f) (Just x) = Just (g (f x))  -- 합성의 정의에 따라
fmap (g . f) (Just x) = fmap g (Just (f x)) -- fmap의 정의에 따라 Just (g (f x)) -> fmap g (Just (f x))로 변경
fmap (g . f) (Just x) = fmap g (fmap f (Just x)) -- fmap의 정의에 따라 이번에는 Just (f x)를 변경
fmap (g . f) (Just x) = (fmap g . fmap f) (Just x) -- 합성의 정의에 따라
```

사이드 이펙트가있는 C++ 스타일의 함수에서는 equational reasoning를 할 수 없습니다.(예제 생략)



#### 7.1.3 Optional

펑터는 하스켈서 표현이 매우 쉽습니다. 하지만 언어가 제네릭과 고차함수를 지원한다면 하스켈이 아니여도 펑터를 정의할 수 있습니다. ~~C++~~ (Swift에서 해보겠습니다.)에서 Maybe와 비슷하게 Optional을 구현해봅시다

```swift
enum Optional<T> {
  case nothing
  case just(_ t: T)
}
```

이 템플릿 유형은 펑터 정의의 한 부분인 타입 매핑을 제공합니다. (타입 T를 Optional<T>로 매핑). 다음으로 함수에 대한 동작을 정의 해 보겠습니다.

```swift
// a -> b함수를 받아 optional<A> -> optional<B>를 반환
func fmap<A, B>(_ mapping: @escaping (A) -> B) -> (Optional<A>) -> Optional<B> {
  guard case let .just(a) = optionalA else {
    return Optional<B>.nothing
  }
  return .just(mapping(a))
}

// 커링 적용 x
func fmap<A, B>(_ optionalA: Optional<A>, mapping: @escaping (A) -> B) -> Optional<B> {
  guard case let .just(a) = optionalA else {
    return Optional<B>.nothing
  }
  return .just(mapping(a))
}

// extension
extension Optional {
    
    func fmap<B>(_ mapping: (T) -> B) -> Optional<B> {
        guard case let .just(a) = self else {
            return .nothing
        }
        return .just(mapping(a))
    }
}
```



#### 7.1.4 Typeclasses

하스켈에서는 추상화된 펑터를 어떻게 다룰까요? -> typeclass 메커니즘을 이용합니다. 타입클래스는 공통 인터페이스를 지원하는 타입 패밀리를 정의합니다. 예를들어 동등 비교를 제공하는 오브젝트들은 아래와 같이 정의됩니다.

```haskell
class Eq a where
  (==) :: a -> a -> Bool
```

이 정의는 타입 a가 == 연산자를 지원하는 경우 Eq 클래스 임을 나타냅니다. (두개의 a 타입을 인자로받아 Bool를 반환하는 연산자). 만일 어떤 특정 타입이 Eq임을 알리고 싶다면 해당 클래스의 인스턴스를 선언하고 == 구현을 제공해야 합니다.

```haskell
-- Float 두개의 product 타입인 Point
data Point = Pt Float Float

-- Point의 equality를 정의
instance Eq Point where
  (Pt x y) == (Pt x' y') = x == x' && y == y'
```

C++이나 Java와 다른점은 Point를 정의할때 Eq 클래스를 구체화할 필요가 없다는 점 입니다. 타입 클래스는 함수를 오버로딩 하기위한 하스켈만의 메커니즘 입니다. 우리는 서로다른 펑터에서 fmap을 오버로딩 해야합니다. 하지만 한가지 문제가 있습니다. 펑터는 타입클래스로 정의되지 않고 타입의 매핑, 타입의 생성자로 정의됩니다. 다행히도 하스켈의 타입클래스는 타입 생성자 및 타입과 같이 작동합니다. 그래서 펑터 클래스를 다음과 같이 정의할 수 있습니다.

```haskell
class Funtor f where
  fmap :: (a -> b) -> f a -> f b
```

위와같이 지정된 타입 시그니처를 가진 함수 fmap이 존재하는 경우 f가 펑터임을 규정합니다. 소문자 f는 타입 변수입니다(a, b 타입 변수와 비슷하게) 컴파일러는 이것의 쓰임을 보고 이것이 타입이 아니라 타입 생성자를 나타낸다고 추론할 수 있습니다.(f a, f b와 같이 다른 타입과의 관계를 이용하여) 따라서 Maybe의 경우처럼 펑터 인스턴스를 선언할 때 타입 생성자를 제공해야 합니다.

```haskell
instance Functor Maybe where
  fmap _ Nothing = Nothing
  fmap f (Just x) = Just (f x)
  
-- Functor, Maybe를 포함한 많은 간단한 데이터 유형의 인스턴스에 대한 정의는 표준 라이브러리 Prelude애 포함되어있습니다.
```

#### ~~7.1.5 Functor in C++ (생략)~~

#### 7.1.6 The List Functor

프로그래밍에서 펑터의 역할에 대하여 더 직관을 얻기위해 더 많은 예를 들어여다 봐야합니다. 다른 타입으로 매개변수화된 모든 타입은 펑터의 후보 입니다. 제네릭한 컨테이너는 저장하는 원소의 타입에 따라 매개변수화 됩니다. 예를 보겠습니다.

```haskell
data List a = Nil | Cons a (List a)
```

우리는 List라는 타입 생성자가 있습니다. 그리고 이는 타입 a를 타입 List a로 매핑합니다. List가 펑터인지 확인하기 위하여 우리는 다음과 같은 리프팅 함수를 정의해야합니다: a -> b => (List a -> List b)

```haskell
fmap :: (a -> b) -> (List a -> List b)
```

List a에서 작동하는 함수는 두개의 List 생성자에 해당하는 두가지 케이스 모두 고려해야 합니다. Nil 케이스는 비교적 간단합니다. 그냥 Nil을 반환하면 됩니다. Cons 케이스는 재귀를 포함하고 있기때문에 좀 까다롭습니다. 일단 우리가 무엇을 해야하는지 다시 생각해봅시다. 우리는 list a가 있고 함수 f는 a -> b로 변환합니다. 그리고 우리는 List b를 만들어야합니다. 분명한점은 함수 f를 이용하여 리스트의 원소들을 변환해야 한다는 점 입니다. Cons 형식으로 헤드 테일이 정의된 비어있지 않은 리스트에서 어떻게 해야할까요? 우선 헤드에 f를 적용하고 꼬리에 리프팅된(fmap이 적용된) f를 적용하면 됩니다.

```haskell
fmap f (Cons x t) = Cons (f x) (fmap f t)
```

우변을 봅시다. fmap f은 list에 적용됩니다. 그리고 기존 list의 정의보다 더 짧습니다. 이것은 결국 더 짧은 형태의 리스트로 재귀하므로 결국 목록 빈목록 혹은 NIl에 도달하게 됩니다. 허나 앞에서 정의하였듯이 Nil에 작동하는 fmap f는 Nil을 반환하므로 재귀가 종료됩니다. 최종적인 결과를 얻기 위하여 헤드 (f x)와 테일을 (fmap f t)를 결합합니다. 정리하여 리스트 펑터의 인스턴스는 다음과 같이 선언할 수 있습니다.

```haskell
instance Functor List where
  fmap _ Nil = Nil
  fmap f (Cons x t) = Cons (f x) (fmap f t)
```

#### 7.1.7 The Reader Functor

지금까지 당신은 펑터가 컨테이너와 같은 역할을 한다고 직관을 얻었을 수 있습니다. 이전까지와는 약간 다른 예를 들어보겠습니다. 타입 a를 a를 리턴하는 함수로 매핑하는 경우에 대하여 고려해 보세요. 우리는 지금까지 함수의 타입에 대해서는 깊게 생각하지 않았습니다. - 완전한 범주적 처리가 다가오고 있습니다. - 하지만 프로그래머로서 우리는 어느정도 이것을 이해하고 있습니다. 하스켈에서 함수 타입은 인자타입, 리턴타임 두가지 타입을 취하여 타입 생성자 -> 를 사용하여 생성됩니다. 이미 a -> b와 같이 중위연산자 유형을 보았지만 이것은 괄호를 사용하여 prefix 형식으로 사용할 수 있습니다: (->) a b

일반 유형의 함수와 마찬가지로 둘 이상의 인자가 있는 타입함수를 부분적으로 사용할 수 있습니다. 따라서 화살표에 하나의 인수 타입만 제공하는 아래와 같은 상황은 다른 인수를 기대한다 할 수 있습니다.

```haskell
(->) a
```

완전한 type a -> b를 생성하려면 b타입이 필요합니다. 위의 상태는 말 그대로 a가 매개 변수화된 모든 유형의 타입 생성자를 정의합니다. 이것도 펑터의 family인지 봅시다. 두가지 유형 매개변수를 다루는 것은 좀 혼란스러울 수 있으므로 이름을 다시 지정해보겠습니다. 위에서 정의한 인라인 펑터의 정의에 따라 인수유형을 r로 리턴타입을 a로 부릅니다. 이 타입 생성자는 a 타입을 받아 (r -> a)로 매핑합니다.  이것이 펑터임을 보이기 위하여 a -> b 함수를  (r -> a) -> (r -> b) 과 같은 형식으로 리프팅 하여야 합니다. 이것은 a 및 b 타입에 대해 작동하는 타입 생성자 (->) r 을 사용하여 형성됩니다. 다음은 이 경우의 fmap 타입 시그니처 입니다.

```haskell
fmap :: (a -> b) -> (r -> a) -> (r -> b)
```

위는 이렇게 해석될 수 있습니다. f :: a -> b함수와 g :: r -> a함수를 이용하여 r -> b 함수를 만들 수 있는가? -> 두개를 합성하면 됩니다. f . g 그렇기 때문에 fmap의 구현은 다음과 같습니다.

```haskell
instance Functor ((->) r) where
  -- fmap :: (z0 -> z1) -> f z0 -> f z1
  -- f가 (-> a) 이면 f z0은 ((-> a) z0) f z1은 ((-> a) z1)
  -- (z0 -> z1) -> ((->) a) z0 -> ((->) a) z1
  -- (z0 -> z1) -> (a -> z0) -> (a -> z1)
  -- 이를 함수 합성 타입과 비교
  -- (.) :: (b -> c) -> (a -> b) -> (a -> c)
  fmap f g = f . g
  -- g :: (r -> a) 의 펑터 fmap f는 
  -- prefix 형식으로 다음과 같이 써도 동일함
  fmap f g = (.) f, g
  -- 그리고 바로 동등함을 보이기 위해 생략도 가능함
  fmap = (.)
```

위와같이 fmap 구현과 (->) r 과 같은 타입 생성자를 reader functor로 부릅니다.(이해 실패.. 🙃 - https://blog.ssanj.net/posts/2018-03-05-functor-applicative-and-monad-instances-for-reader.html)

### 7.2 Functors as Containers

우리는 이미 프로그래밍 언어에서 컨테이너 형식이나 적어도 매게변수화된 타입의 일부를 포함하는 객체와 같은 형식의 펑터를 보았거나. 이와달리 리더 펑터는 좀 특이합니다. 왜나면 우리는 함수를 데이터로 생각하지 않기 때문입니다. 하지만 순수한 함수들은 memorized 될수있고 리턴값또한 룩업테이블과 같다는것을 보았습니다. 테이블은 데이터 입니다. 반대로 하스켈에서는 언어의 게으름 특성으로 인해 리스트와 같이 전형적인 컨테이너가 함수로 구현될 수 있습니다. 무한한 자연수를 상상해보세요. 다음과 같이 단순하게 정의할 수 있습니다.

```haskell
nats :: [Interger]
nats = [1..]
```

첫줄에서 보이는 한쌍의 대괄호는 하스켈의 내장형 리스트 생성자 입니다. 두번째줄에서 대괄호는 리스트 리터럴을 만드는데 사용됩니다. 분명하게 무한한 자연수는 메모리에 담을 수 없습니다. 컴파일러는 요청시 정수를 생성하는 함수로 구현합니다. 하스켈은 데이터와 코드의 구분을 효과적으로 모호하게 합니다. 리스트는 함수로 간주될수고 있고, 테이블에서 입력값에 해당하는 결과값을 매핑해 반환해주는 함수일수도 있습니다. 두번쨰경우 유한범위나 대상이 적은경우 효과적입니다. 하지만 반대의 경우에는 실용적이지 못합니다. 프로그래머로서 우리는 무한을 싫어합니다. 하지만 카테고리이론에서 극한은 밥먹듯이 등장합니다. 그것이 모든 스트링이던 존재할 수 있는 모든 상태이건, 과거의, 현재의 미래의 값이든 우리는 이것을 다룰 수 있습니다. 그래서 펑터 객체(endofunctor에 의해 생성된 타입 객체)가 그것이 물리적으로 존재하지 않더라도 값이나 매개변수화된 타입 값을 포함한다 생각하고 싶습니다. 펑터에 대한 한가지 예로 C++의 std::future 입니다. 이는 특정 시점에 값이 있거나 없음을 뜻합니다. 그리고 이 값에 어세스하고 싶다면 다른 스레드가 이를 기다리는것을 방지할 수 있습니다. 다른 예는 하스켈의 IO 오브젝트 입니다. 이는 유저 인풋과 모니터에 출력되는 값을 뜻합니다. 위 해석에 의해 펑터 객체는 값을 보관하거나 매개변수화된 타입의 값을 보관하고 있습니다. 혹은 이 값을 생산하는 로직을 담고있을 수 있습니다. 우리는 이 값에 접근하는것에 걱정하지 않습니다. - 이것은 매우 선택적이로 펑터 외부의 일입니다. 우리가 오직 관심있는것은 이 값들을 함수를 이용하여 변형시킬 수 있냐는 것 입니다. 만일 값에 접근할 수 있었으면 이 변형의 결과도 접근가능해야 합니다. 만일 그렇지 않다면 우리가 신경쓰는 것은 변형이 올바르게 합성되고 identity function으로 인한 변형이 어떠한 변화도 일으키지 않아야 한다는 것 입니다. 펑터오브젝트가 포함하는 값에 접근하는 것에 얼마나 신경쓰지 않는다는것을 보이기 위해 예를 들어보겠습니다.

```haskell
data Const c a = Const c
```

Const 타입생성자는 c, a 두개의 타입을 받습니다. 화살표 생성자에서 했던것 처럼 우리는 부분적으로 적용하여 펑터를 만들 것 입니다. Const라 불리는 데이타 생성자는 c 타입 하나만 필요합니다. a에 의존이 없습니다. 위의 생성자에 대한 fmap은 다음과 같습니다.

```haskell
fmap :: (a -> b) -> Const c a -> Const c b
```

펑터는 타입 인수를 무시하기 때문에 fmap의 구현은 함수 인자를 무시할 수 있습니다. - 이함수는 하는 역할이 없습니다.

```haskell
instance Functor (Const c) where
  fmap _ (Const v) = Const v
```

```swift
struct Const<C, A> {
  let _v: C
  init(v: C) {
    self._v = v
  }
}

func fmap<C, A, B>(mapping: (A) -> B, const: Const<C, A>) -> Const<C, b> {
  return Const<C, B>(v: const._v)
}
```

이상하게 보이지만 Const 펑터는 많은 생성자에서 중요한 역할을 합니다. 카테고리 이론에서 Δ𝑐 펑터의 특이한 케이스 입니다. 이전에 언급했던것과 같이 블랙홀과 같은 endo-functor 케이스 입니다. 다음에 더 자세히 알아보겠습니다.

### 7.3 Functor Composition

집합에서 집합간의 두 함수를 합성하듯이 카테고리에서 카테고리간 펑터를 합성한다는 것이 가능하다 확신하는것은 어렵지 않습니다. 어떤 대상에 대하여 두 펑터를 합성하는 것은 그들 각자의 오브젝트 매핑을 합성하는 것 입니다. 사상의 경우에도 마찬가지 입니다. 두 펑터 사이를 점핑하여도 항등사상은 항등사상으로 끝납니다. 그리고 사상의 합성은 사상의 합성으로 마무리 됩니다. 특별할건 없습니다. 그중에서도 엔도펑터를 합성하기는 더 쉽습니다. 일전에 다뤘던 maybeTail 함수가 기억나나요? 하스켈의 빌트인되어있는 리스트를 사용하여 구현하면 다음과 같습니다.

```haskell
maybeTail :: [a] -> Maybe [a]
maybeTail [] = Nothing
maybeTail (x:xs) = Just xs
```

maybeTail의 결과 타입은 두가지 펑터의 합성입니다. - a에 대한 Maybe와 []. 펑터들은 모두 각각의 fmap이 내장되어있습니다. 만일 어떤 함수 f를 이 합성된 펑터의 내용물에 적용하려면 어떻게 될까요? 우리는 두 펑터의 결합 레이어를 부수어야합니다. 우리는 fmap을 이용하여 Maybe를 뚫을 수 있지만 단순하게 f를 Maybe의 내부로 보낼 수 없습니다. -> f는 리스트에 대하여 동작하지 않기 때문입니다. 우리는 maybe 내부에 위치한 리스트에 대해 동작하는 (fmap f)을 보내야 합니다. 예를들어 정수 리스트를 요소로 하는 Maybe의 내부 엘리먼트를 제곱하는 방법을 살펴보겠습니다.

```haskell
square x = x * x

mis :: Maybe [Int]
mis = Just [1, 2 , 3]

mis2 = fmap (fmap square) mis
-- mis(Maybe) 의 원소(List)에 fmap square전달, fmap square를 f라 하면
-- Just (f [1, 2, 3]), f [1,  2, 3]은 fmap square [1, 2, 3] list펑터 내부의 원소에 square 함수 적용
```

컴파일러는 타입추론 이후에 바깥쪽에 위치한 fmap이 Maybe 인스턴스의 구현을, 내부의 fmap은 리스트 펑터의 구현을 사용해야한다는 것을 알게될 것 입니다. 위의 코드가 다음과 같이 쓰여질 수 있다는 점이 바로 와닿지는 않을것 입니다.

```haskell
mis2 = (fmap . fmap) square mis
-- 두번째 fmap은 square(Int -> Int)를 [Int] -> [Int]로 변환
-- 이후 첫번째 펑터는 ([Int] -> [Int]) -> (Maybe[Int] -> Maybe[Int])
-- (([Int] -> [Int]) -> (Maybe[Int] -> Maybe[Int])) . ((Int -> Int) -> ([Int] -> [Int]))
-- 최종적으로 이 함수가 mis에 적용
```

따라서 두 펑터의 합성은 각 펑터에 해당하는 fmap의 합성과 같습니다. 다시 카테고리로 돌아와서 펑터의 합성은 결합법칙이 성립한다는 것은 매우 분명합니다.(대상의 매핑도 사상의 매핑도 다 결합법칙이 성립) 그리고 대상을, 사상을 각각 자기 자신으로 매핑하는 identity 펑터가 있다는 것도 알수있습니다. 이렇기 때문에 펑터는 일부 카테고리에서의 사상과 동일한 속성을 모두 지닙니다. 그러나 이것은 어떤 범주 이여야 할까요? **대상이 범주이고 사상이 펑터인 범주 이여야합니다.** 이것은 카테고리들의 카테고리 입니다. 하지만 집합에서 그랬듯이(모든 집합의 집합이 불가능하다) 모든 카테고리의 카테고리는 자기 자신도 포함해야하고 이경우에 다시 역설에 빠지게 됩니다. 그렇기에 모든 small 카테고리의 카테고리는 Cat이라 부릅니다. 작은 카테고리는 집합보다 큰것이 아니라 대상들이 집합을 형성하는 범주 입니다.(A small category is one in which objects form a set, as opposed to something larger than a set.) 다시한번 상기하자면 카테고리 이론에서 무한한 셀수없는 집합조차도 작은것(small)로 간주됩니다. 우리는 여러 수준에 추상화에서 반복되는 동일한 구조를 인식할 수 있기때문에 저자는 이점을 언급하고 싶다고 합니다. 나중에는 펑터가 카테고리를 형성하는것을 알아보겠습니다.
