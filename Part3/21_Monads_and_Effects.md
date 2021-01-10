## 21. Monads and Effects



이제 모나드가 무엇인지 압니다 - 이는 장식함수들의 합성법 입니다. - 매우 흥미로운 질문은 이 장식함수가 함수형 프로그래밍에서 그렇게 중요한지 입니다. 지금까지 예로 Writer 모나드를 보았습니다.  Writer 모나드는 여러 펑션 콜사이에서 로그를 만들고 축적합니다. 순수하지 않은 방식으로(전역상태에 접근하여) 로그를 만드는 이 문제는 Writer를 이용하여 순수하게 해결되었습니다.



### 21.1 The Problem

아래는  [Eugenio Moggi’s seminal pape](https://core.ac.uk/download/pdf/21173011.pdf)에서 가져온 이와 유사한 문제의 목록입니다. 모든 이 문제들은 전통적으로 순수하지 않은 함수를 이용하여 해결할 수 있습니다.

- Partiality: 종료할 수 없는 연산(Computations that may not terminate)
- Nondeterminism: 여러 결과를 리턴할 수 있는 연산(Computations that may return many results)
- Side effects: 상태에 접근하거나 수정하는 연산
  - Read-Only state or the environment
  - Write-Only state, or a log
  - Read/wrtite state
- Exceptions: 실패가능한 Partial 함수(Partial functions that my fail)
- Continuations: 프로그래밍의 상태를 저장하고 필요한 경우에 복원하는 능력(Ability to save state of the program and then restore it on demand)
- Interactive Input
- Interactive Output

놀라운 사실은 이 모든 문제들이 동일한 트릭을 이용하여 해결될 수 있습니다: 장식함수를 이용하는 것 입니다. 물론 모든 경우에 장식함수는 다를것입니다.

이 단계에서 장식함수가 모나딕할 필요가 없다는 점을 유의하여야합니다. 모나드가 필요한것은 우리가 합성을 고집할떄 단일 장식함수를 더 작은 단위로 나눌 수 있다는 것 입니다. 다시말해서 각각의 장식함수가 다르기 때문에 모나딕 합성은 각기 다르게 구현될것 입니다. 하지만 전체적인 패턴은 동일합니다. 이 패턴은 매우 단순합니다: 합성은 결합법칙을 만족해야하고 동등한 identity가 있어야합니다.

다음 섹션은 하스켈 예제가 엄청 많을것 입니다. 스킵하는거는 맘대로 하세요



### 21.2 The Solution

우선 Writer 모나드를 분석하는것으로 시작해 봅시다. 주어진 입력에 대하여 특정한 아웃풋을 만들어내는 순수 합수로 시작했었습니다. 이 함수를 장식함수로 대체하였는데 원래 함수의 아웃풋이 string 과 함꼐 페어링되는것으로 바뀌었습니다. 이는 로깅 문제를 위한 우리의 해결책 이였습니다.

일반적으로 monolithic한 해결책을 원하지 않기 때문에 여기서 멈출수는 없습니다. 로그를 만들어내는 한 함수를 더 작은 로그를 만들어내는 함수로 분해해야 합니다.이 작은 함수들의 합성은 우리를 모나드의 개념으로 이끕니다.

매우 놀라운점은 함수의 리턴타입을 장식하는 동일한 패턴이 일반적으로 순수성을 포기하여야하는 다양한 문제에 적용된다는 것입니다. 앞서 이야기했던 문제 목록을 살펴보고 각 문제에 적용되는 장식법을 구분해 보죠.



#### 21.2.1 Partiality

종료될수 없음을 뜻하는 “lifted”된 타입으로 모든 함수의 리턴타입을 바꿀 수 있습니다. 여기서 리피팅된 타입은 원래 타입의 모든 값을 포함함과 동시에 "bottom" 발류를 포함합니다. Bool을 예를들어보자면 Bool은 True, False 값 두개를 포함하는 집합입니다. 리프팅된 Bool은 3가지 값으로 구성되고 리프팅된 Bool을 반환하는 함수는 True, False 혹은 영원히 실행할 수 없음을 리턴합니다.

재미있는점은 하스켈과 같은 레이지한 언어에서 종료될 수 없는 함수는 실제로 값을 반환합니다. 그리고 이 값은 다음 함수로 전달될 수 있습니다. 이 특별한 값은 바텀이라 부릅니다. 이 값이 명백하게 필요하기 전까지(예를들어 패턴 매칭이나 결과물을 만들어 내는 경우) 이 값은 프로그램의 중단 없이 계속 전달될 수 있습니다. 하스켈 함수들은 잠재적으로 non-terminating 하기 때문에, 하스켈의 모든 타입들은 리프팅되있다 가정할 수 있습니다. 이는 Set 대신에 종종 하스켈의 (리프팅된) 타입들의 범주인 Hask를 이야기했던 이유입니다. 그러나 Hask가 실제 카테고리라는점은 확실하지 않습니다.(see this [Andrej Bauer post2](http://math.andrej.com/2016/08/06/hask-is-not-a-category/)).



#### 21.2.2 Nondeterminism

함수가 만일 다양한 결과를 리턴할 수 있다면 동시에 이값들을 한번에 리턴할수도 있을것입니다. 의미상으로는 non-deterministic 함수는 결과들의 리스트를 리턴하는 함수와 동등합니다. 이것은 lazy garbage-collected 언어에서 많은 의미가 있습니다. 예를들어 필요한 모든것이 하나의 값이라면 리스트의 헤드를 취하면 되고 테일은 평가되지 않을것 입니다. 랜덤 발류가 필요하다면 리스트의 n 번째 값을 취하는 랜덤넘버 제너레이터를 이용하면 됩니다. Laziness는 심지어 결과값들의 무한한 리스트를 리턴할 수 있게 해줍니다.

하스켈의 nondeterministic computations의 구현인 리스트 모나드에서 join 은 concat으로 정의되어있습니다. join이 컨테이너의 컨테이너를 평평하게 만드는것을 기억하세요. concat은 리스트의 리스트를 하나의 리스트로 연결합니다. 그리고 return은 싱글 리스트를 만들어 냅니다.

```haskell
instance Monad [] where
	join = concat
	return x = [x]
```

리스트 모나드의 바인드 연산자는 다음과 같은 일반 공식으로 제공됩니다. 이와같은 경우에 join이후에 fmap이 따라옵니다.

```haskell
as >>= k = concat (fmap k as)
```

그 자체로 리스트를 만들어내는  k 함수는 as의 모든 원소에 적용됩니다. 결과타입은 리스트의 리스트 이고 이는 concat을 이용하여 플래튼 해 집니다.

프로그래머적 관점에서 리스트를 다루는것은 매우 쉽습니다. 예를들어 루프 안에서 non-deterministic 함수를 호출하거나 이터레이터를 리턴하는 함수를 구현하는 것 입니다.

 non-deterministic creatively의 좋은 예는 게임 프로그래밍 입니다. 예를들어 인간과 대적하는 체스게임을 예로 들때 상대편의 다음수가 무엇일지 예상하는것은 불가능 합니다. 하지만 모든 가능한 수를 계산하여 리스트형식으로 반환하는것은 가능합니다. 비슷하게 non-deterministic 파서는 파싱 가능한 모든 표현을 만들어 냅니다.

non-deterministic 하다는것을 리스트를 리턴하는것으로 해석했음에도 리스트 모나드의 적용은 더욱 광범위 합니다. 이는 리스트를 생성하는 계산을 함께 연결하는 것이 명령형 프로그램에서 사용하는 반복(루프)를 완벽하게 기능적으로 대체하기 때문입니다. 단일 루프는 종종 루프의 바디를 리스트의 각 요소에 적용하는 fmap을 사용하여 다시 작성할 수 있습니다.(A single loop can be often rewritten using fmap that applies the body of the loop to each element of the list.) 리스트 모나드의 표기법을 이용하여 복잡합 네스티드 루프를 대체하는데 사용할 수 있습니다.

제가 제일 좋아하는 예는 Pythagorean 트리플을 만드는 프로그램 입니다. - 직각 삼각형의 변을 형성 할 수 있는 양의 정수 트리플을 찾는거 말입니다.

```haskell
triples = do
	z <- [1..]
	x <- [1..z]
	y <- [x..z]
	guard (x^2 + y^2 == z^2)
	return (x, y, z)
```

첫줄에서 z는 무한한 양수 리스트를 의미합니다. 그리고 x는 1과 z 사이 (유한)리스트 [1..z]의 원소입니다. 마지막으로 y는 x와 z 사이의 리스트의 원소입니다. 이 제안헤 1<=x<=y<=z를 만족하는 3 숫자가 있습니다.  guard 함수는 Bool 표현을 받아 unit 리스트를 반환 합니다.

```haskell
guard :: Bool -> [()]
guad True = [()]
guard False = []
```

이 함수는(MonadPlus라 불리는 더 큰 클래스의 멤버) Pythagorean triples 가 아닌 값을 거르는데 사용합니다. 실제로 bind 연산자의 구현을 보면(혹은 관련된 >>) 이를 알아차릴 수 있습니다. 반대로 엠티리스트가 아닌 값이 반환된다면(이경우에 [( )]) 바인드는 return (z, y, z) continuation를 콜할것이고 Pythagorean triple로 확인된 싱글톤 리스트를 만들어 냅니다. 이러한 모든 싱글톤 리스트는 최종(무한한) 결과를 만들기 위하여 둘러싸는 바인딩에 의해 연결됩니다. 물론 triples의 콜러는 모든 리스트를 사용할 수 없습니다. 하지만 하스켈은 레이지하기 때문에 상관 없습니다.

리스트 모나드와 do 표기법을 이용하여 복잡한 삼중루프 구문이 매우 단순해 졌습니다. 이를 리스트 comprehension를 이용하여 더 단순화 할 수 있습니다.

```haskell
triples = [(x, y, z) | z <- [1..],
	x <- [1..z],
	y <- [x..z],
	x^2 + y^2 == z^2]
```

이것은 리스트 모나드를 위한 syntactic sugar 입니다(엄밀히 말해서 MonadPlus)

제너레이터 및 코루틴을 가장하여 다른 함수적 또는 명령형 언어에서 유사한 구문을 볼 수 있습니다.



#### 21.2.3 Read-Only State

외부 상태나 환경에 read-only로 접근하는 함수는 environment를 추가적인 어규먼트로 받아들이는 함수로 대체될 수 있습니다. 순수한 함수 (a, e) -> b(여기서 e는 environment를 뜻함) 처음 보면 클라이슬리 화살표처럼 보이지는 않습니다. 하지만 커링으로 이를 a -> (e -> b)로 바꾸면 장식함수가 우리의 친구 리더 펑터가 됨을 발견할 수 있습니다.

```haskell
newtype Reader e a -> Reader (e -> a)
```

Reader를 반환하는 함수를 작은 실행가능 구문을 생성하는 것으로 해석할 수 있습니다: environment가 주어지면 원하는 결과가 반환되는 것 말입니다. 다음은 위와 같은 동작을 위한 헬퍼펑션 runReader 입니다.

```haskell
 runReader :: Reader e a -> e -> a
 runReader (Reader f) e = f e
```

이는 환경값에 따라서 다른 결과를 만들어낼것 입니다. 

두 함수가 모두 리더를 반환하고 리더 그 자체로 순수함을 주목하세요/

리더모나드의 바인드 구현을 위하여 환경 e를 받아 b를 반환하는 함수를 만들어야 한다는 점을 주목하세요.

```haskell
ra >>= k = Reader (\e -> ...)
```

람다의 내부에서 ra가 a를 만들어내도록 동작을 실행할 수 있습니다.

```haskell
rx >>= k = Reader (\e -> let a = runReader ra e
	in ...)
```

이러면 새로운 동작 rb를 얻기 위하여 a를 연속되는 k에 넘겨줄 수 있습니다.

```haskell
ra >>= k = Reader (\e -> let a = runReader ra e
	rb = k a
	in ...)
```

마지막으로 rb 동작에 e를 이용하여 실행시킬 수 있습니다.

```haskell
ra >>= k = Reader (\e -> let a = runReader ra e
	rb = k a
	in runReader rb e)
```

return 의 구현은 e를 입력받아 이를 무시하고 변경되지 않은 값을 반환하는 동작을 만들어 할 수 있습니다.

```haskell
instance Monad (Reader e) where
	ra >>= k = Reader (\e -> runReader (k (runReader ra e)) e)
	return x = Reader (\e -> x)
```



#### 21.2.4 Write-Only State

처음 예를 들었던것 처럼 장식은 Writer 펑터로 주어질 수 있습니다.

```haskell
newtype Writer w a -> Writer (a, w)
```

이를 완설시키기 위하여 데이터 스트럭처를 언패킹하는 runWriter라는 헬퍼 펑션이 있습니다.

```haskell
runWriter :: Writer w a -> (a, w)
runWriter (Writer (a, w)) = (a, w)
```

위에서 보앗던것 처럼 Writer를 함성가능하게 만들기 위하여 w는 모노이드 이여야 합니다. 다음은 Writer를 위한 모나드 인스턴스의 바인드 연산자 구현입니다.

```haskell
instance (Monoid w) => Monad (Writer w) where
	(Writer (a, w)) >>= k = let (a', w') = runWriter(k a)
		in Writer (a' w 'mappend' w')
	return a = Writer (a, mempty)
```



#### 21.2.5 State

상태에 대하여 read/write 할 수있는 함수는 Reader와 Writer의 합성입니다. 이를 상태를 추가적인 어규먼트로 받아 값과 상태 페어를 결과값으로 리턴하는 것으로 생각할 수 있습니다; (a, s) -> (b, s) 커링을 이용하여 클라이슬리 화살표를 형성하게 해봅시다. a -> (s -> (b, s)), 이경우 장식은 State 펑터로 추상화 됩니다.

```haskell
newtype State s a = State (s -> (a, s))
```

다시한번 클라이슬리 화살표가 동작을 리턴한다는것을 볼 수 있습니다. 그리고 이는 헬퍼펑션에 의하여 실행될 수 있습니다.

```haskell
runState :: State s a -> s -> (a, s)
runState (State f) s = f s
```

초기 상태가 다른경우에 다른 결과를 만들어 내는 것은 물론 최종 상태도 다르게 될 것 입니다.

State 모나드의 바인드 연산 구현은 각 단계에서 올바른 상태를 전달하는데 주의를 기울여야한다는 점을 제외하고는 Reader의 경우와 매우 유사합니다. 

```haskell
sa >>= k = State (\s -> let (a, s') = runState sa s
	sb = k a
	in runState sb s')
	
-- full instance
instance Monad (State s) where
	sa >>= k = State (\s -> let (a, s') = runState sa s
		in runState (k a) s')
	return a = State (\s -> (a, s))
```

상태를 조작하는데 사용할 수 있는 두개의 헬퍼 클라이슬리 화살표도 있습니다.

```haskell
get :: State s s
get = State (\s -> (s, s))

-- replace new state
put :: s -> State s ()
put s' = State (\s -> ((), s'))
```



#### 21.2.6 Exceptions

명령형 함수에서 에러를 스로우하는것은 실제로 partial 입니다. - 이는 어규먼트의 특정 값에 정의되지 않은 함수를 뜻합니다. 순수한 함수에서 예외를 구현하는 가장 단순한 방법은 Maybe functor 를 이용하는 것 입니다. partial 함수는 그것이 가능할때 Just를 가능하지 않을떄는 Nothing을 반환하는 것으로 확장할 수 있습니다. 실패의 원인에 대한 추가 정보를 같이 반환하고 싶다면 Either 펑터를 이용하면 됩니다(첫번째 타입, 예를들어 String, 이 고정)

다음은  Maybe 모나드 인스턴스 입니다.

```haskell
instance Monad Maybe where
	Nothing >>= k = Nothing
	Just a >>= k = Just k a
	return a = Just a
```

Maybe에 대한 모나드 합성은 예외가 발생했을때 중단됩니다.(연속적으로 k가 호출되지 않음) 이는 예외가 하는 동작과 일치합니다.



#### 21.2.7 Continuations

"우리를 호출하지마 우리가 호출할께" 와 같은 상황을 잡 인터뷰 이후에 많이 겪어보았을 것 입니다. 바로 답을 주는것 이전에 결과값과 함께 호출되어하는 핸들러를 제공하여야합니다. 이 스타일은 특히 호출시점에 결과값을 알지 못하는 상황에 유용합니다. 예를들어 이는 다른 스레드나 remote 웹서버에서 얻어오는 값 일 수 있습니다. 이경우에 클라이슬리 화살표는 핸들러를 받아들이는 함수를 리턴합니다. 그리고 이는 나머지 연산을 나타냅니다.

```haskell
data Cont r a = Cont ((a -> r) -> r)
```

핸들러 a -> r은 결과값을 만들어내기 위해 최종적으로 호출됩니다. continuation은 결과 타입에 의해 매개변수화 됩니다.(실제로 이것은 일종의 상태표시기 입니다.)

클라이슬리 화살표에 의해 반환된 작업을 실행하는 헬퍼 펑션이 있습니다. 핸들러를 가져와서 연속으로 전달합니다.

```haskell
runCont :: Cont r a -> (a -> r) -> r
runCont (Cont k) h = k h
```

연속의 합성은 어렵기로 악명높습니다. 그렇기에 모나드를 이용하여 특별히 do 표기법을 이용하여 다루는것은 큰 장점입니다.

바인드 연산자의 구현을 생각해봅시다. 우선 제거된 시그니처를 보겠습니다.(First let’s look at the stripped down signature:)

```haskell
(>>=) :: ((a -> r) -> r) ->
	(a -> (b -> r) -> r) ->
	((b -> r) -> r)
	
-- ka = (a -> r) -> r
-- kab = (a -> (b -> r) -> r)
-- kb = (b -> r) -> r
```

우리의 목표는 핸들러 (b -> r)을 받아 r을 만들어내는 것 입니다. 그렇기에 아래와같이 시작할 수 있습니다.

```haskell
ka >>= kab = Cont (\hb -> ...)
```

람다 안쪽에서 나머지 연산을 나타내는 적절한 핸들러와 함께 ka를 호출하고자 할 것 입니다. 이 핸들러를 람다로 구현해보죠

```haskell
runCont ka (\a -> ...)
```

이 경우에 나머지 연산은 먼저 kab를 a와 함께 콜하고 hb를 이것에 넘겨 kb의 동작을 만드는 것 입니다.

```haskell
runCont ka (\a -> let kb = kab a
	in runCont kb hb)
```

위에서 보듯이 continuations은 안에서 밖으로 합성됩니다. 마지막 핸들러 hb는 제일 내부 레이어의 연산에 의해 호출됩니다.

```haskell
instance Monad (Cont r) where
	ka >>= kab = Const (\hb -> runcCont ka (\a -> runCont (kab a) hb))
	return a = Cont (\ha -> ha a)
```



#### 21.2.8 Interactive Input

이는 가장 까다로운 문제 이며 많은 혼란의 원인이 됩니다. 명백하게 키보드 인풋의 캐릭터 타입을 반환하는 getChar 타입은 순수하지 않습니다. 하지만 만약에 컨테이너에 둘러쌓이 캐릭터를 반환한다면 어떨까요? 이 컨테이너에서 캐릭터를 추출할 수 없는한 우리는 이 함수가 순수하다고 주장할 수 있습니다. 이러면 getChar를 호출할때마다 매번 같은 컨테이너가 반환될 것 입니다. 개념적으로 이 컨테이너는 방생 가능한 모든 캐릭터들을 포함합니다.

만일 당신이 양자 메커니즘에 친숙하다면 이 유사함을 이해하는데 큰 어려움을 겪지는 않을것 입니다. 이것은 마치 슈레딩거의 고양이가 들어가있는 박스와 같습니다. 유일하게 다른점은 이 상자를 열거나 안에 보관된 값을 꺼낼 수 없다는 점 입니다. 이 박스는 특별한 빌트인 IO 펑터를 이용하여 정의됩니다. 우리의 예제 getChar에서 이는 클라이슬리 화살표로 선언될 수 있습니다.

```haskell
getChar :: () -> IO Char
```

(실제로 유닛타입을 입력받는 함수는 리턴타입에 해당하는 값을 골라내는 것이라 생각할 수 있기 때문에 다음과 같이 단순화 될 수 있습니다. getChar :: IO Char)

IO가 펑터 이기때문에 fmap을 이용하여 그 내용물을 조작할 수 있습니다. 그리고 또 펑터이기 때문에 캐릭터 타입 말고 다른 타입들도 내용물로 담을 수 있습니다. 하스켈에서 이 접근법은 IO를 모나드로 다룰때 빛을 발할것 입니다. 이는 IO 객체를 만들어내는 클라이슬리 화살표를 합성할 수 있음을 뜻합니다.

틀라이슬리 합성은 IO 객체의 내용물을 꺼내는것을 가능하게 한다고 생각할 수 있습니다. 실제로 getChar를 캐릭터를 입력으로 하는 클라이슬리 화살표와 합성할 수 있습니다. 정수로 바꾼다고 해봅시다. 주목할점은 두번째 클라이슬리 화살표는 정수를 (IO Int)로 리턴한다는 점 입니다. 반복해서 말하자면 이제는 발생가능한 모든 int로 끝나게 되는것 입니다. 슈레딩거의 고양이는 박스 밖으로 절대 나오지 않습니다. 만일 IO 모나드 안으로 들어간다면 빠져나올 방법은 없습니다. runState나 runReader와 같이 IO 모나드를 위한 runIO는 없습니다.

그러면 IO 객체를 반환하는 클라이슬리 화살표의 결과값  합성하는거 말고 어떻게 다루어야 할까요? 이는 main에서 반환할 수 있습니다. 하스켈에서 main은 다음과 같은 시그니쳐를 지닙니다.

```haskell
main :: IO ()
```

그리고 이를 클라이슬리 화살표로 생각해도 좋습니다.

```haskell
main :: () -> IO ()
```

이러한 관점에서 하스켈 프로그래은 IO 모나드에서 하나의 큰 클라이슬리 화살표 일뿐 입니다. 모나딕 합성을 이용하여 작은 클라이 화살표들로 부터 이를 합성할 수 있습니다. 결과인 IO 객체로(IO action 이라고도 함)로 작업을 하는것은 런타임 시스템에 달려있습니다.

화살표 자체는 순수하다는 점에 주목하세요. 지저분한 작업은 시스템이 합니다. 최종적으로 main에서 IO action이 반환되며 실행될때 유저 인풋을 읽거나, 파일을 수정하거나, 메세지를 프린팅 하거나, 디스크를 포맷하는것과 같은 지저분한 작업들이 처리됩니다. 하스켈 프로그램은 절대 손을 더럽히지 않습니다.

물론 하스켈은 레이지 하기 때문에 main은 거의 즉시 반환되고 더러운 작업들은 즉시 시작됩니다. 순수 계산의 결과가 요청시에 요청되고 평가되는것은 IO 작업을 실행하는 동안 입니다. 따라서 실제 프로그램 실행은 순수 하스켈 코드와 더러운 시스템 코드의 인터리빙(interleaving) 입니다.

더 대조적이기는 하지만 수학적 모델로 완벽하게 이해되는 IO 모나드를 해석하는 다른 방식이 있습니다. 이는 전체 우주를 프로그램에서 하나의 객체로 다룹니다. 개념적으로 명령형 프로그래밍에서 universe를 외부 상태 객체로 다룬다는 것에 주목하세요. 따라서 IO를 수행하는 절차는 해당 객첵와 상호 작용하기때문에 사이드 이펙트가 발생합니다. Universe의 상태를 읽고 수정할 수 있기 때문입니다.

상태를 함수형 프로그래밍에서 어떻게 다루는지는 이미 보았습니다. - State 모나드를 사용합니다. 다른 단순한 상태와는 다르게 Universe는 표준 데이터 구조로 묘사하기 힘듭니다. 하지만 그것과 집적적으로 상호작용하지 않는 이상 그럴 필요도 없습니다. Real World라는 타입이 있다 가정하고 런타임에서 이에 해당하는 객체를 제공할 수 있다고 가정하면 됩니다. IO 액션은 다음과 같은 함수입니다.

```haskell
type IO a = ReadWorld -> (a, RealWord)

-- or in terms of the State monad:
type IO = State RealWorld
```

하지만 이 모나드에 대한 >=> 나 return는 언어에 빌트인 되어있어야 할것입니다.





#### 21.2.9 Interactive Output

인터엑티브한 아웃풋을 추상화하기 위하여 동일한 IO 모나드가 이용됩니다. RealWorld는 기기에 가능한 모든 아웃풋을 포함한다 가정합니다. 왜 하스켈에서 출력 함수를 바로 호출할수는 없는지 궁금할것 입니다. 예를들어 아래의 방식을 선호하는지 말입니다.

```haskell
putStr :: String -> IO ()
-- rather than the simple:
putStr :: String -> ()
```

두가지 이유가 있습니다: 하스켈은 레이지하기 때문에 출력(여기서는 단위객체)이 아무것도 사용되지 않는 함수를 호출하지 않습니다. 그리고 레이지 하지 않더라도 이러한 호출의 순서를 자유롭게 변경하여 출력이 왜곡될 수 있습니다. 하스켈에서 두 함수에 순차적인 실행을 부여하는 방법은 데이터 디펜던시를 이용해서 입니다. 연속되는 함수의 인풋타입은 다른 함수의 리턴 타입에 달려있습니다. IO 작업간에 RealWorld를 전달하면 시퀀싱이 적용됩니다.

```haskell
main :: IO ()
main = do
	putStr "Hello "
	putStr "World!"
```

개념적으로 위의 프로그램에서 “World!”를 출력하는 동작은 이미 "Hello "가 출력되고 있는 Universe를 인풋으로 받습니다. 이 결과물은 “Hello World!”가 화면에 나타나는 새로운 Universe 입니다.

















