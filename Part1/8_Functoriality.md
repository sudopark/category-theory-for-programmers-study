# Chapter 8: Functoriality — 펑터성

> **핵심 개념**: 대수적 데이터 타입(ADT)은 합과 곱의 조합으로 만들어지는데, 합과 곱 자체가 **쌍펑터(Bifunctor)**이므로 ADT도 자동으로 펑터가 된다. 또한 함수의 인자 위치에 따라 **공변(covariant)**과 **반변(contravariant)**이 구분되며, 둘을 합친 것이 **프로펑터(Profunctor)**이다.

---

## 이 장의 핵심 질문

Ch7에서 펑터를 배웠다: 타입 생성자 + fmap이 있고 펑터 법칙을 만족하면 펑터다. 그런데 `Maybe`, `List`, `Tree` 같은 ADT마다 매번 fmap을 직접 구현하고 법칙을 증명해야 할까?

이 장의 답은 **"아니오"**다. ADT는 곱과 합의 조합인데, 곱과 합이 쌍펑터(두 인자 펑터)이므로 **ADT는 자동으로 펑터가 된다.** 이것을 보이기 위해:

1. 먼저 **쌍펑터**를 정의한다 — 타입 인자가 두 개인 펑터
2. 곱 `(a, b)`과 합 `Either a b`가 쌍펑터임을 확인한다
3. 쌍펑터의 한쪽을 고정하면 일반 펑터가 됨을 보인다
4. ADT = 곱과 합의 조합이므로, 타입 파라미터에 대해 **자동으로 펑터**라는 결론을 얻는다

후반부에서는 펑터의 방향에 대해 더 파고든다: **공변**(방향 보존), **반변**(방향 뒤집힘), 그리고 둘을 합친 **프로펑터**.

---

## 쌍펑터 (Bifunctor)

Ch7에서 본 펑터는 타입 인자가 **하나**인 타입 생성자에 대한 것이었다 — `Maybe a`, `[a]`, `(->) r a` 등. 그런데 `(a, b)`나 `Either a b`는 타입 인자가 **두 개**다. 이런 것도 펑터일 수 있을까?

**쌍펑터(Bifunctor)**는 두 개의 인자를 받는 펑터다. 카테고리 이론에서는 두 카테고리의 **곱 카테고리(product category)** C × D에서 E로의 펑터로 정의된다. 곱 카테고리란 두 카테고리의 대상과 사상을 쌍으로 묶은 것이다:

- **대상**: C와 D의 대상의 쌍 (a, b)
- **사상**: C와 D의 사상의 쌍 (f, g) where f :: a → a', g :: b → b'
- **합성**: 각 성분별로 합성

프로그래밍에서는 간단히 "두 타입 인자에 대해 각각 함수를 적용할 수 있는 것"으로 이해하면 된다:

```haskell
class Bifunctor f where
    bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
    bimap g h = first g . second h
    
    first :: (a -> c) -> f a b -> f c b
    first g = bimap g id
    
    second :: (b -> d) -> f a b -> f a d
    second h = bimap id h
```

`bimap`은 두 함수를 동시에 적용하고, `first`와 `second`는 각각 한쪽만 변환한다. Ch7의 `fmap`이 하나의 함수를 적용했다면, `bimap`은 **두 함수를 양쪽에 동시에** 적용하는 것이다.

### 곱은 쌍펑터

페어 `(a, b)`에서 양쪽 값을 각각 변환할 수 있다:

```haskell
instance Bifunctor (,) where
    bimap f g (x, y) = (f x, g y)

-- 예시
bimap show (+1) ("hello", 3) = (show "hello", 4)
--     ↑     ↑                    ↑            ↑
--   첫째변환 둘째변환         첫째에 적용   둘째에 적용
```

```swift
func bimap<A, B, C, D>(
    _ f: (A) -> C, _ g: (B) -> D, _ pair: (A, B)
) -> (C, D) {
    return (f(pair.0), g(pair.1))
}
```

### 쌍대곱(Either)도 쌍펑터

Either는 값이 하나만 들어있으므로, 해당하는 쪽의 함수만 적용된다:

```haskell
instance Bifunctor Either where
    bimap f _ (Left x)  = Left (f x)     -- 왼쪽이면 f 적용
    bimap _ g (Right y) = Right (g y)    -- 오른쪽이면 g 적용

-- 예시
bimap (+1) show (Left 3)   = Left 4       -- f만 적용
bimap (+1) show (Right 3)  = Right "3"    -- g만 적용
```

---

## 쌍펑터에서 펑터로: 한쪽 고정

쌍펑터는 타입 인자가 두 개인데, 프로그래밍에서 만드는 펑터는 보통 타입 인자가 하나다 (`Maybe a`, `[a]` 등). 어떻게 연결될까?

**쌍펑터의 인자 하나를 고정하면, 나머지 인자에 대해 일반 펑터가 된다.** 예를 들어:

- `Either () a` — `Either`는 쌍펑터, 첫째를 `()`로 고정 → `a`에 대해 펑터 → 이것이 `Maybe a`
- `(String, a)` — `(,)`는 쌍펑터, 첫째를 `String`으로 고정 → `a`에 대해 펑터 → 이것이 `Writer a`

bimap에서 고정된 쪽은 `id`를 넣는 것과 같다:

```haskell
-- Either () a에서 a만 변환
bimap id f (Right x) = Right (f x)    -- = fmap f (Just x)

-- (String, a)에서 a만 변환
bimap id f (s, x) = (s, f x)         -- = fmap f (s, x)
```

## ADT는 자동으로 펑터가 된다

이제 핵심 결론에 도달한다.

1. Ch6에서 배웠다: **모든 ADT는 합(Either)과 곱(Pair)의 조합이다**
2. 이 장에서 확인했다: **합과 곱은 쌍펑터이고, 한쪽을 고정하면 일반 펑터가 된다**
3. Ch7에서 배웠다: **펑터의 합성은 펑터다**

따라서:

> **ADT는 타입 파라미터에 대해 자동으로 펑터가 된다.** 즉, `a` 자리에 함수를 흘려보낼 수 있다 — 구조(모양)는 그대로 유지하면서 안에 담긴 값의 타입만 변환할 수 있다.

`Maybe`로 확인해보자:

```haskell
Maybe a = Nothing | Just a
        = Either () a          -- Ch6에서 배운 동형
```

`Either`는 쌍펑터이고, 첫 번째 인자 `()`는 고정이므로, 두 번째 인자 `a`에 대해 일반 펑터가 된다.

### Tree — 더 복잡한 ADT도 마찬가지

Tree는 합과 곱이 재귀적으로 조합된 ADT다:

```haskell
data Tree a = Leaf a | Node (Tree a) (Tree a)
--          = Either a (Tree a, Tree a)
--            ↑ 합(쌍펑터)  ↑ 곱(쌍펑터)
```

합과 곱으로 분해되고, 각각이 쌍펑터이므로 `a`에 대해 fmap을 만들 수 있다:

```haskell
instance Functor Tree where
    fmap f (Leaf a)   = Leaf (f a)           -- a에 f 적용
    fmap f (Node l r) = Node (fmap f l) (fmap f r)  -- 재귀적으로 a에 f 적용
```

```swift
indirect enum Tree<A> {
    case leaf(A)
    case node(Tree<A>, Tree<A>)
    
    func fmap<B>(_ f: (A) -> B) -> Tree<B> {
        switch self {
        case .leaf(let a): return .leaf(f(a))
        case .node(let l, let r): return .node(l.fmap(f), r.fmap(f))
        }
    }
}
```

트리의 **구조(모양)**는 그대로이고, Leaf에 담긴 **값의 타입만** 바뀐다. 이것이 "a 자리에 함수를 흘려보낸다"는 의미다.

> Haskell에서는 `deriving Functor` 확장을 사용하면 컴파일러가 자동으로 fmap을 생성해준다 — ADT의 구조를 보고 합과 곱을 따라 함수를 흘려보내는 코드를 만들어내는 것이다.

### Writer — 곱 쌍펑터의 한쪽 고정

```haskell
type Writer a = (String, a)

-- (,)는 쌍펑터, 첫째를 String으로 고정 → 둘째 a에 대해 펑터
-- Haskell에서 (,) String は (String, a) 형태가 된다
instance Functor ((,) String) where
    fmap f (s, a) = (s, f a)
    -- String(로그)은 그대로 두고, a만 변환

-- 예시
fmap (+1) ("log: started", 3) = ("log: started", 4)
--         ↑ String 그대로      ↑ a만 변환됨
```

---

## 공변 펑터와 반변 펑터

### 공변 펑터 (Covariant Functor)

지금까지 본 일반적인 펑터. 사상의 방향을 **보존**한다:

```
f :: a → b  ⟹  fmap f :: F a → F b
-- a에서 b로 가는 함수가 있으면, F a에서 F b로 가는 함수도 있다
-- 방향이 같다: a→b 이면 Fa→Fb
```

`Maybe`, `List`, `Reader` 등 Ch7에서 본 펑터는 모두 공변이다.

### 반변 펑터 (Contravariant Functor)

사상의 방향을 **뒤집는** 펑터. 카테고리 이론에서는 반대 카테고리 C^op에서의 공변 펑터에 해당한다:

```
f :: a → b  ⟹  contramap f :: F b → F a
-- a에서 b로 가는 함수가 있으면, F b에서 F a로 가는 함수가 있다
-- 방향이 반대: a→b 이면 Fb→Fa
```

```haskell
class Contravariant f where
    contramap :: (b -> a) -> f a -> f b
```

이게 언제 자연스러울까? 프로그래밍에서 익숙한 예시로 보자.

**예시: Predicate (조건 검사)**

```haskell
newtype Predicate a = Predicate (a -> Bool)

-- "짝수인가?"는 Int에 대한 조건
isEven :: Predicate Int
isEven = Predicate (\n -> n `mod` 2 == 0)

-- 문자열의 길이가 짝수인지 검사하고 싶다면?
-- String → Int 함수(length)를 "앞에 붙이면" 된다
isEvenLength :: Predicate String
isEvenLength = contramap length isEven
-- = Predicate (\s -> isEven (length s))
```

`length :: String -> Int`는 String→Int 방향인데, 결과는 `Predicate Int`에서 `Predicate String`으로 **반대 방향**이다. 함수의 **입력 쪽**에 변환을 끼워넣기 때문에 방향이 뒤집히는 것이다.

**Op 펑터** — 같은 원리의 일반화:

```haskell
newtype Op r a = Op (a -> r)

instance Contravariant (Op r) where
    contramap f (Op g) = Op (g . f)
    -- f :: b -> a, g :: a -> r
    -- g . f :: b -> r
    -- 결과: Op r a → Op r b (a가 b로, 방향 뒤집힘)
```

### 함수 타입에서의 분산(variance)

함수 타입 `a → b`에서 위치에 따라 분산이 다르다:

```haskell
-- 반환 타입(출력) 위치: 공변 — fmap으로 변환 (뒤에 합성)
fmap show getPort = show . getPort
--                  출력을 변환 ↑

-- 인자 타입(입력) 위치: 반변 — contramap으로 변환 (앞에 합성)
contramap length isEven = isEven . length
--                 입력을 변환 ↑
```

직관: **출력 쪽을 바꾸면 방향이 같고(공변), 입력 쪽을 바꾸면 방향이 뒤집힌다(반변).**

---

## 프로펑터 (Profunctor)

방금 함수 타입 `a → b`가 입력(a)에 대해 반변, 출력(b)에 대해 공변이라는 걸 봤다. 이 두 성질을 합친 것이 **프로펑터(Profunctor)**다: 첫 번째 인자에 대해 **반변**, 두 번째 인자에 대해 **공변**인 쌍펑터. 카테고리 이론에서는 C^op × D → Set 형태의 펑터로 정의된다.

```haskell
class Profunctor p where
    dimap :: (a' -> a) -> (b -> b') -> p a b -> p a' b'
    dimap f g = lmap f . rmap g
    
    lmap :: (a' -> a) -> p a b -> p a' b   -- 왼쪽(입력): 반변
    lmap f = dimap f id
    
    rmap :: (b -> b') -> p a b -> p a b'   -- 오른쪽(출력): 공변
    rmap g = dimap id g
```

`bimap`이 양쪽 다 **같은 방향**으로 변환했다면, `dimap`은 왼쪽은 **역방향**, 오른쪽은 **정방향**으로 변환한다.

**함수 타입 `(->)`는 프로펑터의 대표적 예시:**

```haskell
instance Profunctor (->) where
    dimap f g h = g . h . f
    -- f :: a' -> a (앞에 합성, 반변 — 입력을 변환)
    -- h :: a -> b  (원래 함수)
    -- g :: b -> b' (뒤에 합성, 공변 — 출력을 변환)
    -- 결과: a' -> b'
```

구체적인 예시:

```haskell
-- 원래 함수: Int를 받아서 Int를 돌려줌
double :: Int -> Int
double x = x * 2

-- dimap으로 양쪽을 변환:
-- 입력: String → Int (length, 앞에 붙임)
-- 출력: Int → String (show, 뒤에 붙임)
dimap length show double :: String -> String
-- = show . double . length
-- = \s -> show (double (length s))

-- "hello" → length → 5 → double → 10 → show → "10"
```

직관: 함수 `h :: a → b`의 양 끝에 **어댑터**를 연결하는 것이다. 입력 쪽에 변환기 `f`를 끼우고, 출력 쪽에 변환기 `g`를 끼워서, 원래 함수가 다루지 못하던 타입도 처리할 수 있게 만든다.

---

## 정리

- **쌍펑터(Bifunctor)**: 두 인자를 받는 펑터. `bimap`으로 양쪽 동시 변환
- **곱 `(a, b)`과 합 `Either a b`는 쌍펑터**
- **ADT는 자동으로 펑터**: 합과 곱의 조합이므로, 타입 파라미터에 대해 펑터성 보존
- **공변(Covariant)**: 사상 방향 보존. `fmap`. 반환 타입 위치
- **반변(Contravariant)**: 사상 방향 뒤집힘. `contramap`. 인자 타입 위치
- **프로펑터(Profunctor)**: 첫 인자 반변 + 둘째 인자 공변. `dimap f g h = g . h . f`
- 함수 타입 `a → b`는 a에 대해 반변, b에 대해 공변인 프로펑터
