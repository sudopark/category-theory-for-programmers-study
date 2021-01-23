## 23. Comonads



이제 모나드를 이해하였고 이제는 쌍대성의 이점을 살려 반대의 카테고리에서 화살표를 뒤집어서 코모나드를 얻을 차례입니다.

기초부터 다시 시작합시다. 모나드는 클라이슬리 화살표 합성에 관한것 입니다.

```haskell
a -> m b
```

m은 모나드이고 펑터 입니다. m이 뒤집어진 형태의 w을 사용하여 co-클라이슬리 화샆요의 타입으로 이용할 수 있습니다.

```haskell
w a -> b
```

코 클라이슬리 화살표에 대응되는 피쉬 오퍼레이터는 다음과 같습니다.

```haskell
(=>=) :: (w a -> b) -> (w b -> c) -> (w a -> c)
```

코 클라이슬리 화살표가 카테고리를 형성하게하기 위하여 다음과같은 코 클라이슬리 화살표의 아이덴티티가 필요하고 이를 extract라 부릅니다.

```haskell
extract :: w a -> a
```

이는 return과 쌍대입니다. 물론 좌측, 우측 아이덴티티에 대하여 결합법칙을 부여하여야합니다. 이 모든것을 합쳐 하스켈에서 코모나드를 다음과 같이 정의할 수 있습니다.

```haskell
class Functor w => Comonad w where
(=>=) :: (w a -> b) -> (w b -> c) -> (w a -> c)
extract :: w a -> a
```

실제로 곧 살펴보겠지만 약간 다른 primitives를 사용합니다. 

그럼 프로그래밍에서 코모나드는 어떤용도로 쓰입니까?



### 23.1 Programmin with Comonads

모나드와 코모나드를 합성해봅시다. 모나드는 return을 이용하여 값을 컨테이너 안으로 집어넣습니다. 내장된 값에 접근할 수 있는 방법은 제공하지 않습니다. 물론 모나드로 구현된 몇몇 데이터 스트럭처는 이들의 컨텐츠에 어세스하는 기능을 제공하긴 하지만 이것은 특별한 경우 입니다. 모나드에서 값을 빼낼 수 있는 공통적인 인터페이스는 없습니다. 그리고 이것을 IO 모나드가 내부를 노출하지 않으면서 자기 자신을 제공하는것을 통하여 보았습니다.

반대로 코모나드는 컨테이너에서 하나의 값을 추출하는 기능을 제공합니다. 이는 값을 넣는것을 제공하지는 않습니다. 만일 코모나드를 컨테이너라 생각한다면 이안에 값은 미리 채워져있고 그 값을 꺼내는 기능을 제공한다 할 수 있습니다.

클라이슬리 화살표가 값을 받아 장식된 결과를 만들어내듯 - 이는 context와 함께 장식합니다 - 코 클라이슬리 화살표는 값과 함께 모든 컨텍스트를 받아 결과를 만들어 냅니다. 이는 켄텍스트에 따른 계산?(contextual computation.)의 구현 입니다.( It’s an embodiment of contextual computation.)



### 23.2 The Product Comonad

리더 모나드를 기억하세요. 리더 모나드를 읽기만 가능한 환경 e를 필요로하는 경우를 해결하기 위해 사용했었습니다. 이와같은 계산은 다음과 같은 순수함수로 나타낼 수 있습니다.

```haskell
(a, e) -> b
```

그리고 커링을 이용하여 이를 클라이슬리 화살표로 바꿀 수 있습니다.

```haskell
a -> (e -> b)
```

원래의 함수는 코 클라이슬리 화살표입니다. 이의 아규먼트를 좀더 편리한 펑터 형태로 손봐보죠.

```haskell
data Product e a = Prod e a deriving Functor
```

합성하려는 화살표와 동일한 환경을 사용하게하여 합성연산자를 쉽게 정의할 수 있습니다.

```haskell
(=>=) :: (Prod e a -> b) -> (Prod e b -> c) -> (Prod e a -> c)
f =>= g = \(Prod e a) -> let b = f (Prod e a)
c = g (Prod e b)
in c
```

그리고 extract의 구현은 단순이 환경을 무시합니다.

```haskell
extract (Prod e a) = a
```

놀랍지 않게도 product comonad를 사용하여 리더 모나드와 똑같은 계산을 수행할 수 있습니다. 이경우에 환경에 대한 코모나드적 구현은 더 자연스럽습니다. - 이는 context에 따른 계산의 정신을 따릅니다. 반면에 모나드는 do 노테이션의 syntactic sugar로 부터 얻어집니다.

리더 모나드와 프로덕트 코모나드관의 관계를 더 깊게 파고든가면 사실상 리더 모나드는 프로덕트 모나드의 우측 adjoint 입니다. 그러나 일반적으로 코모나드는 모나드와는 다른 계산 개념을 다룹니다. 나중에 이 예를 더 볼것입니다.

product comonad를 튜플과 records를 포함하는 임의의 프로덕트 타입으로 일반화하는것은 쉽습니다.



### 23.3 Dissecting the Composition

듀얼라이제이션 과정을 계속하며 모나딕 바인드와 조인으로 가봅시다. 모나드에서 사용하였던 방법을 반복해봅시다. 여기서 피쉬 오퍼레이터를 해부했었는데 이 과정을 통해 더 많은 깨달음을 얻을 수 있습니다.

합성 연산자가 w a를 받아 c를 만들어내야 한다는 점에서 부터 시작해 봅시다. c를 만들어내는 유일한 방법은 w b 타입의 인수에 두번째 함수를 적용하는 것 입니다.

```haskell
(=>=) :: (w a -> b) -> (w b -> c) -> (w a -> c)
f =>= g = g ...
```

하지만 g에 입력할 수 있는 타입 w b는 어떻게 만들 수 있을까요? 이를 위한 함수 f :: w a -> b가 있습니다. 그리고 바인드의 듀얼 버전을 정의하여 w b 를 만들게 할 수 있는데 이는 extend라 부릅니다.

```haskell
extend :: (w a -> b) -> w a -> w b

-- 그리고 extend를 이용하여 합성을 구현할 수 있습니다.
f =>= g. = g . extend f
```

다음으로 extend를 해부할 수 있습니까? w a -> b를 w a 인수에 바로 적용하면 안되냐 생각할 수 있습니다. 하지만 그러면 결과로 나오는 b를 w b로 바꾸는 방법이 없다는점을 알게될 것 입니다. 코모나드가 값을 리프팅하는 방식을 제공하지 않는다는 점을 기억하세요. 이점에서 비슷하게 모나드를 만들기 위하여 fmap을 사용하였습니다. 이제 여기서 fmap을 사용할 수 있는 유일한 방법은 처리할 수 있는 w (w a) 타입이 있는 경우 입니다. w a 를 w (w a)로 만들 수 있다면 이는 join의 듀얼 이고 이를 duplicate라 부릅니다.

```haskell
duplicate :: w a -> w (w a)
```

모나드의 정의와 비슷하게 이제 모나드에 대한 3가지 정의가 있습니다: 코 클라이슬리 화살표, extend, duplicate. 다음은 Control.Comonad 라이브러리에서 직접 가져온 하스켈의 정의 입니다.

```haskell
class Functor w => Comonad w where
extract :: w a -> a
duplicate :: w a -> w (w a)
duplicate = extend id
extend :: (w a -> b) -> w a -> w b
extend f  = fmap f . duplicate

-- extend in terms of duplicate
-- f :: (w a) -> b
-- g :: (w b) -> c
-- fmap f = w(w a) -> w b
-- extend f wa = fmap f (duplicate wa)
-- point-free => extend f = fmap f . duplicate
```

기본적으로 extend의 구현이 duplicate에 따라 구현되며 반대의 경우도 마찬가지 이므로 둘 중 하나만 구현하면 됩니다.

이 함수들을 바탕으로 얻을 수 있는 직관은 다음의 아이디어를 바탕으로 합니다. 일반적으로 코모나드는 타입 a에 해당하는 값들로 채워진 컨테이너로 생각할 수 있습니다.(product comonad는 하나의 값만 있는 특수한 케이스 입니다.) extract를 통하여 어세스할 수 있는 "current" value라는 개념이 있습니다. 코 클라이슬리 화살표는 current value에 집중한 연산을 수행하지만 이는 둘러싼 모든 값들에 접근할 수 있어야 합니다. Conway’s game of life을 생각해보세요. 모든 셀은 값을 포함하고 있고(보통 True 혹은 False). 게임에 해당하는 코모나드는 "현재" 셀에 초첨을 맞춘 셀들의 격자 일것입니다.

그러면 duplicate는 무엇을 할까요? 이는 코모나드 컨테이너 w a 를 받아 컨테이너의 컨테이너 w (w a)를 만들어 냅니다. 이 아이디어는 각 컨테이너가 모두 w a 안에 다른 a에 집중하고 있다는 점 입니다. (게임에서 비유 생략)

extend를 봅시다. 이는 코 클라이슬리 화살표와 a들이 담긴 코모나드 컨테이너 w a를 입력받습니다. 그리고 이 모든 a들을 b로 바꿉니다. extend는 포커스를 a에서 다른 a로 이동하고 코 클라이슬리 화살표를 차례로 적용합니다. (게임 비유 생략)





### 23.4 The Stream Comonads

컨테이너의 하나의 원소에서 다른곳으로 포커스를 옮기는 이 절차는 무한한 스트림 예제로 설명할 수 있습니다. 이는 엠티 생성자가 없다는 점을 제외하고는 list와 같습니다

```haskell
data Stream a = Cons a (Stream a)

instance Functor Stream where
fmap f (Cons a as) = Cons (f a) (fmap f as)
```

스트림의 포커스는 첫번쨰 원소이고 다음은 extract의 구현입니다.

```haskell
extract (Cosn a _) = a
```

duplicate는 각각의 다른 원소에 초점을 맞춘 스트림의 스트림을 만듭니다. 

```haskell
duplicate (Cons a as) = Cons (Cons a as) (duplicate as)
```

첫번째 원소는 원래의 스트림이고 두번쨰 원소는 본래 스트림의 테일 입니다. 세번쨰 원소는 그것의 테일이고 이것은 계속됩니다.

```haskell
instance Comonad Stream where
extract (Cons a _) = a
duplicate (Cons a as) = Cons (Cons a as) (duplicate as)
```

이는 스트림을 보는 매우 함수적인 방법 입니다. 명령형 언어에서 스트림을 다룰때 advance와 같이 한 포지션 쉬프팅 시키는 것으로 시작했을것 입니다. 여기서 duplicate는 한번에 모든 시프팅된 스트림을 만들어 냅니다. 하스켈의 레이지함이 이를 가능하게 하고 심지어 바람직(desirable) 하게 만듭니다. 물론 스트림을 실용적으로 만들기 위하여 advance와 비슷한것을 만들 수 있습니다.

```haskell
tail :: Stream a -> Stream a
tail (Cons a as) = as
```

하지만 이는 코모나드의 인터페이스는 아닙니다.

디지털 신호처리를 경험한적이 있다면 스트림에 대한 코 클라이슬리 화살표는 디지털 필터를 의미한다는 것을 알아차릴 수 있을것입니다. 그리고 extend는 필터링된 스트림을 만들어 냅니다.

간단한 예로 이동평균필터를 만들어 보죠. 다음은 스트림의 n개의 엘리먼트의 합을 구하는 함수 입니다.

```haskell
sumS :: Num a => Int -> Stream a -> a
sumS n (Cons a as) = if n <= 0 then 0 else a + sumS (n - 1) as
```

다음은 스트림의 n번째 원소까지 평균을 구하는 함수 입니다.

```haskell
average :: Factorial a => Int -> Stream a -> a
average n stm = (sumS n stm) / (fromIntegral n)
```

average n의 부분적인 적용은 코-클라이슬리 화살표 입니다. 이제 이를 extend를 이용하여 전체 스트림에 적용할 수 있습니다.

```haskell
movingAvg :: Factorial a => Int -> Stream a -> Stream a
movingAvg n = extend (average n)
```

결과는 이동평균필터가 적용된 스트림 입니다.

스트림은 단방향 일차원 코모나드 입니다. 이를 쉽게 쌍방이나 2, 3차원르로 확장하는것은 쉽습니다.



### 23.5 Comonad Categorically

코모나드를 카테고리 이론에서 정의하는것은 듀얼리티에 대한 간단한 연습입니다. 모나드에서 엔도펑터 T와 두개의 자연변환 η, μ로 시작했었습니다. 코모나드에서는 이를 뒤집으면 됩니다.

> 𝜀∷𝑇→𝐼
>
> 𝛿∷𝑇→𝑇2

이 변환의 컴포넌트는 extract와  duplicate와 동일합니다. 코모나드 법칙은 모나드 법칙의 미러 이미지입니다. 

수반에서 모나드를 정의하였고 쌍대성은 이를 뒤집습니다. 좌측 인접은 우측 인접이 되고 반대도 마찬가지 입니다. R.L 합성이 모나드를 정의하기 때문에 L.R은 코모나드를 정의해야 합니다. 수반의 counit은 다음과 같습니다.

> 𝜀∷𝐿∘𝑅→𝐼

이는 위에서 정의한 코모나드의 𝜀와 동일합니다. - 혹은 컴포넌트적으로 하스켈의 extract. 물론 수반의 unit도 사용할 수 있습니다.

> 𝜂∷𝐼→𝑅∘𝐿

 𝐿∘𝑅 사이에 𝑅∘𝐿을 넣으면 𝐿∘𝑅∘𝐿∘𝑅가 만들어 집니다. T에서 T^2를 만드는 것은 𝛿를 정의하고 이는 코모나드의 정의를 완료합니다.

그리고 모나드는 모노이드임을 보았었습니다. 이 문장의 쌍대 버전을 위해 우리는 코모노이드가 필요합니다. 근데 코모노이드는 무엇입니까? 대상이 하나인 카테고리라는 모노이드의 원래 정의에는 쌍대의 개념이 없습니다. 모든 엔도모피즘을 뒤집으면 다른 카테고리를 얻게 됩니다. 하지만 이는 모나드에 대한 접근 이였습니다. 우리는 모노이드를 모노이달 카테고리의 대상으로 더 일반적인 정의를 하였었습니다. 이 구조는 다음의 두가지 사상을 바탕으로 합니다.

> 𝜇∷𝑚⊗𝑚→𝑚
>
> 𝜂∷𝑖→𝑚

이 사상들의 역은 모노이달 카테고리에서 코모노이드를 만들어 냅니다.

> 𝛿∷𝑚→𝑚⊗𝑚 
>
> 𝜀∷𝑚→𝑖

하스켈에서는 다음과 같이 코모노이드를 정의할 수 있습니다.

```haskell
class Comonoid m where
split :: m -> (m, m)
destroy :: m -> ()
-- destroy는 아규먼트를 무시하기에
destroy _ = ()
-- split는 단지 함수 쌍 입니다.
split x = (f x, g x)
```

모노이드 유닛 법칙과 쌍대인 코모노이드 버전은 다음과 같습니다.

```haskell
lambda . bimap destroy id . split = id
rho . bimap id destroy . split = id
```

여기서 lambda와 rho는 각각 왼쪽, 오른쪽 unitor 입니다. 정의들을 합쳐 다음을 얻을 수 있습니다.

```haskell
lambda (bimap destroy id (split x))
= lambda (bimap destroy id (f x, g x))
= lambda ((), g x)
= g x
```

이는 g = id를 제공합니다. 그리고 두번쨰 법칙은 f = id로 확장됩니다. 결론적으로 다음과 같습니다.

```haskell
split x = (x, x)
```

이는 하스켈에서(일반적으로 카테고리 Set에서) 모든 대상은 코모노이드 입니다.

운좋게도 코모노이드를 정의하는 모노이달 카테고리가 더 있습니다. 이는 엔도펑터의 카테고리 입니다. 그리고 이는 모나드가 엔도펑터의 카테고리에서 모노이드인것과 같이,

> 코모나드는 엔도펑터 카테고리에서 코모노이드입니다.



### 23.6 The Store Comonad

코모나드의 중요한 다른점은 이는 상태모나드의 쌍 이라는 점 입니다. 이는 costate comonad 혹은 store comonad라 불립니다. 

상태 모나드를 익스포넨셜을 정의하는 수반에 의해 만들어 진다는 것을 보았습니다:

> L z = z x s
>
> R a = s => a

코 스테이트 코모나드를 정의하기 위하여 동일한 수반을 이용할 것 입니다. 코모나드는 L . R 합성으로 정의됩니다.

> L ( R a) = (S => a) x s

이를 하스켈로 옮겨 좌측의 프로덕트 펑터와 우측의 리더 펑터로 시작해 봅시다. 리더 펑터 이후의 프로덕트 펑터의 합성은 다음과 같습니다:

```haskell
data Store s a = Store (s -> a) s
```

대상 a에서 얻어진 수반의 코유닛은 다음과 같은 사상 입니다.

> 𝜀𝑎 ∷ ((𝑠 ⇒ 𝑎) × 𝑠) → 𝑎

혹은 하스켈에서는 다음과 같습니다.

```haskell
counit (Prod (Reader f) s) = f s
```

이는 위에서 보았던 extract가 됩니다.

```haskell
extract (Store f s) = f s
```

수반의 유닛은 다음과 같고 data constructor를 부분적으로 적용하여 다음과 같이 쓸 수 있습니다.

```haskell
unit :: a -> Reader s (Product a s)
unit a = Reader (\s -> Prod a s)

Store f :: s -> Store f s
```

이는 수평적 합성에 해당하는 𝛿 혹은 duplicate과 같습니다.

> 𝛿∷𝐿∘𝑅→𝐿∘𝑅∘𝐿∘𝑅 
>
> 𝛿=𝐿∘𝜂∘𝑅

프로덕트 펑터인 가장 왼쪽에 있는 L을 통해 𝜂을 몰래 빠져나가야 합니다. 이는 쌍의 왼쪽 구성 요소에서 𝜂 또는 Store f와 함께 작동하는 것을 의미합니다.(이는 프로덕트에 대한 fmap의 동작)

```haskell
duplicate (Store f s) = Store (Store f) s
```

(𝛿, L, R에 대한 공식이 컴포넌트가 아이덴티티 사상인 아이덴티티 자연변환을 나타냅을 기억하세요.)

아래는 정의된 Store 코모나드의 정의 입니다.

```haskell
instance Comonad (Store s) where
extract (Store f s) = f s
duplicate (Store f s) = Store (Store f) s
```

Store의 Reader 부분은 s 타입의 키가 지정된 as들의 컨테이너로 생각할 수 있습니다. 예를들어 만일 s가 Int라면 Reader Int는 무한한 쌍방향 as의 스트림입니다. Store는 이 컨테이너와 키 타입의 값과 쌍을 이룹니다. 예를들어 Reader Int a 는 Int와 쌍을 이룹니다. 이경우에 extract는 이 정수를 무한한 스트림에서의 인덱스로 사용합니다. Store의 두번째 컴포넌트를 현재 위치로 생각할 수도 있습니다.

이 예를 계속들어보면 duplicate는 Int로 인덱싱된 새로운 무한한 스트림을 만들어 냅니다. 이 스트림은 원소로 스트림을 포함합니다. 특히, 현재 포지션에 대하여 원래의 스트림을 포함합니다. 하지만 만일 다른 Int(+ 혹은 -)를 키로 사용한다면 새로운 인덱스로 시프팅된 스트림을 얻어낼 수 있습니다.

Duplicated 된 Store에 extract를 적용하면 원래의 Store를 얻게 됩니다(사실상 코모나드의 identity raw는 extract . duplicate = id임을 나타냅니다.)

Store comonad는 Lens 라이브러리에서 매우 중요한 이론을 담당합니다. 개념적으로 Store s a 코모나드로서 특정 데이터 타입을 포커스(렌즈 처럼) 할 수 있는 a는 타입으로 s는 index로 이용하는 서브구조의 개념을 캡슐화 합니다. 

```haskell
a -> Store s a
-- 위와 같은 타입의 함수는 아래의 함수 쌍과 같습니다.
set :: a -> s -> a
get :: a -> s
```

만일 a가 프로덕트 타입 이라면 a의 수정된 버전을 반환하는 동안에 내부에서 s 타입의 필드를 설정하여 set을 구성 할 수 있습니다. 비슷하게. 마찬가지로 a에서 s 필드의 값을 읽기 위하여 get을 구현할 수 있습니다. 이 아이디어는 다음 섹션에서 알아볼것입니다.







