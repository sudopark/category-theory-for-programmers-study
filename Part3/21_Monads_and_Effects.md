# Chapter 21: Monads and Effects — 모나드와 효과

> **핵심 개념**: 모나드는 순수 함수형 프로그래밍에서 **사이드 이펙트(side effect)**를 모델링하는 일반적 메커니즘이다. 부분성(partiality), 비결정론(nondeterminism), 읽기/쓰기 상태, 예외, 연속(continuation) 등 다양한 계산 효과가 각각 다른 모나드로 표현된다.

---

## 왜 모나드로 효과를 표현하는가?

순수 함수는 `a → b`다. 하지만 현실의 프로그램은:
- 실패할 수 있고 (부분 함수)
- 여러 결과를 낼 수 있고 (비결정론)
- 외부 상태를 읽거나 쓰고
- 예외를 던지고
- I/O를 수행한다

모나드는 이런 효과를 **반환 타입의 장식(embellishment)**으로 인코딩한다: `a → m b`. 순수 함수의 합성 가능성을 유지하면서 효과를 다룰 수 있다.

---

## 주요 모나드들

### 1. 부분성: Maybe / Optional

함수가 결과를 반환하지 못할 수 있는 경우:

```haskell
safeDiv :: Double -> Double -> Maybe Double
safeDiv _ 0 = Nothing
safeDiv x y = Just (x / y)

-- 합성: 앞 단계가 실패하면 전체가 실패
safeDiv 10 2 >>= \r -> safeDiv r 0
-- = Nothing (0으로 나누기 실패 전파)
```

### 2. 비결정론: List

하나의 입력에 여러 출력이 가능한 경우:

```haskell
-- 피타고라스 삼조: 비결정적 탐색
triples :: [(Int, Int, Int)]
triples = do
    z <- [1..]
    x <- [1..z]
    y <- [x..z]
    guard (x*x + y*y == z*z)
    return (x, y, z)
-- [(3,4,5), (5,12,13), (6,8,10), ...]
```

List 모나드의 bind는 각 선택지에 대해 함수를 적용하고 결과를 합친다 (concatMap). 이것은 **탐색 공간의 탐색**이다.

### 3. 로깅: Writer

순수하게 로그를 누적하는 경우:

```haskell
type Writer w a = (a, w)

-- w는 모노이드여야 함 (누적 연산 + 빈 로그)
tell :: w -> Writer w ()
tell w = ((), w)

logProcess :: Int -> Writer [String] Int
logProcess x = do
    tell ["Starting"]
    let y = x * 2
    tell ["Doubled"]
    return y
```

### 4. 상태: State

읽기/쓰기 가능한 상태를 순수하게 다루는 경우:

```haskell
type State s a = s -> (a, s)

get :: State s s
get s = (s, s)

put :: s -> State s ()
put s _ = ((), s)

-- 예: 카운터
increment :: State Int ()
increment = do
    n <- get
    put (n + 1)
```

State 모나드는 "상태를 받아 (결과, 새 상태)를 반환하는 함수"를 합성한다.

### 5. 환경 읽기: Reader

읽기 전용 환경에 접근하는 경우:

```haskell
type Reader r a = r -> a

ask :: Reader r r
ask r = r

-- 예: 설정 읽기
greet :: Reader String String
greet = do
    name <- ask
    return ("Hello, " ++ name)
```

Reader 모나드는 공유 환경을 암묵적으로 전달한다.

### 6. 예외: Either

실패 시 에러 정보를 포함하는 경우:

```haskell
type Result a = Either String a

parseAge :: String -> Result Int
parseAge s = case reads s of
    [(n, "")] | n >= 0 -> Right n
    _ -> Left ("Invalid age: " ++ s)
```

### 7. 연속: Continuation

계산의 나머지(continuation)를 다루는 경우:

```haskell
type Cont r a = (a -> r) -> r

-- CPS 변환과 관련
-- 요네다 보조정리와 깊은 연결
```

---

## 효과의 합성

여러 효과를 결합해야 하는 경우 **모나드 변환자(Monad Transformer)**를 사용한다:

```haskell
-- Reader + Writer + State를 결합
type App a = ReaderT Config (WriterT [Log] (State AppState)) a
```

모나드 변환자는 모나드를 쌓아 올려 여러 효과를 동시에 다룰 수 있게 한다.

---

## 정리

- **모나드 = 효과의 추상화**: 순수 함수로 사이드 이펙트를 모델링
- **Maybe**: 부분성 (실패 가능한 계산)
- **List**: 비결정론 (여러 결과 탐색)
- **Writer**: 로깅 (출력 누적, 모노이드 기반)
- **State**: 가변 상태 (s → (a, s) 형태의 함수 합성)
- **Reader**: 환경 읽기 (공유 설정의 암묵적 전달)
- **Either**: 예외 (에러 정보 포함)
- **Cont**: 연속 (계산의 나머지를 다룸)
- **모나드 변환자**: 여러 효과를 합성하여 쌓기
