# Chapter 8: Functoriality — 펑터성

> **핵심 개념**: 대수적 데이터 타입(ADT)은 합과 곱의 조합으로 만들어지는데, 합과 곱 자체가 **쌍펑터(Bifunctor)**이므로 ADT도 자동으로 펑터가 된다. 또한 함수의 인자 위치에 따라 **공변(covariant)**과 **반변(contravariant)**이 구분되며, 둘을 합친 것이 **프로펑터(Profunctor)**이다.

---

## 쌍펑터 (Bifunctor)

**쌍펑터(Bifunctor)**는 두 개의 인자를 받는 펑터다. 두 카테고리의 **곱 카테고리(product category)** C × D에서 E로의 펑터로 정의된다.

곱 카테고리 C × D의 구조:
- **대상**: C와 D의 대상의 쌍 (a, b)
- **사상**: C와 D의 사상의 쌍 (f, g) where f :: a → a', g :: b → b'
- **합성**: 각 성분별로 합성

쌍펑터는 두 인자 각각에 대해 펑터적으로 동작한다:

```haskell
class Bifunctor f where
    bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
    bimap g h = first g . second h
    
    first :: (a -> c) -> f a b -> f c b
    first g = bimap g id
    
    second :: (b -> d) -> f a b -> f a d
    second h = bimap id h
```

`bimap`은 두 함수를 동시에 적용하고, `first`와 `second`는 각각 한쪽만 변환한다.

### 곱은 쌍펑터

```haskell
instance Bifunctor (,) where
    bimap f g (x, y) = (f x, g y)
```

```swift
func bimap<A, B, C, D>(
    _ f: (A) -> C, _ g: (B) -> D, _ pair: (A, B)
) -> (C, D) {
    return (f(pair.0), g(pair.1))
}
```

### 쌍대곱(Either)도 쌍펑터

```haskell
instance Bifunctor Either where
    bimap f _ (Left x)  = Left (f x)
    bimap _ g (Right y) = Right (g y)
```

---

## 펑터적 대수적 데이터 타입

ADT는 합(Either)과 곱(Pair)의 조합이다. 합과 곱이 모두 쌍펑터이므로, **ADT를 이루는 타입 파라미터에 대해 자동으로 펑터가 된다.**

예: `Maybe a = Nothing | Just a`는 `Either () a`와 동형이고, `()`는 상수이므로 a에 대해 펑터다.

### Tree 펑터 예시

```haskell
data Tree a = Leaf a | Node (Tree a) (Tree a)

instance Functor Tree where
    fmap f (Leaf a)   = Leaf (f a)
    fmap f (Node l r) = Node (fmap f l) (fmap f r)
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

> Haskell에서는 `deriving Functor` 확장을 사용하면 컴파일러가 자동으로 fmap을 생성해준다.

### Writer 펑터

```haskell
type Writer a = (a, String)

-- 곱 쌍펑터에서 String을 고정하면 나머지 인자에 대해 펑터
instance Functor ((,) String) where
    fmap f (s, a) = (s, f a)
```

---

## 공변 펑터와 반변 펑터

### 공변 펑터 (Covariant Functor)

지금까지 본 일반적인 펑터. 사상의 방향을 보존한다:

```
f :: a → b  ⟹  fmap f :: F a → F b
```

### 반변 펑터 (Contravariant Functor)

사상의 방향을 **뒤집는** 펑터. 이것은 반대 카테고리 C^op에서의 공변 펑터에 해당한다:

```
f :: a → b  ⟹  contramap f :: F b → F a   -- 방향이 반대!
```

```haskell
class Contravariant f where
    contramap :: (b -> a) -> f a -> f b
```

**대표 예시: Op 펑터** — 함수의 인자 위치

```haskell
newtype Op r a = Op (a -> r)

instance Contravariant (Op r) where
    contramap f (Op g) = Op (g . f)
    -- f :: b -> a, g :: a -> r 이므로 g . f :: b -> r
```

직관: 함수 `a → r`이 있을 때, `b → a`를 **앞에 합성**하면 `b → r`이 된다. 입력 타입이 b에서 a로 갔지만(역방향), 결과 타입은 `Op r b`가 된다.

### 함수 타입에서의 분산(variance)

함수 타입 `a → b`에서:
- **반환 타입 b** 위치: **공변** (fmap으로 변환 — 뒤에 합성)
- **인자 타입 a** 위치: **반변** (contramap으로 변환 — 앞에 합성)

---

## 프로펑터 (Profunctor)

첫 번째 인자에 대해 **반변**, 두 번째 인자에 대해 **공변**인 쌍펑터를 **프로펑터(Profunctor)**라 한다. 형식적으로 C^op × D → Set 형태의 펑터다.

```haskell
class Profunctor p where
    dimap :: (a' -> a) -> (b -> b') -> p a b -> p a' b'
    dimap f g = lmap f . rmap g
    
    lmap :: (a' -> a) -> p a b -> p a' b   -- 왼쪽: 반변
    lmap f = dimap f id
    
    rmap :: (b -> b') -> p a b -> p a b'   -- 오른쪽: 공변
    rmap g = dimap id g
```

**함수 타입 `(->)`는 프로펑터의 대표적 예시:**

```haskell
instance Profunctor (->) where
    dimap f g h = g . h . f
    -- f :: a' -> a (앞에 합성, 반변)
    -- h :: a -> b  (원래 함수)
    -- g :: b -> b' (뒤에 합성, 공변)
    -- 결과: a' -> b'
    
    lmap f h = h . f       -- 앞에 합성 (반변)
    rmap g h = g . h       -- 뒤에 합성 (공변)
```

직관: 함수 `h :: a → b`를 양쪽에서 감싼다 — 입력 변환 `f`를 앞에, 출력 변환 `g`를 뒤에. 마치 파이프라인의 양 끝을 어댑터로 연결하는 것과 같다.

---

## 정리

- **쌍펑터(Bifunctor)**: 두 인자를 받는 펑터. `bimap`으로 양쪽 동시 변환
- **곱 `(a, b)`과 합 `Either a b`는 쌍펑터**
- **ADT는 자동으로 펑터**: 합과 곱의 조합이므로, 타입 파라미터에 대해 펑터성 보존
- **공변(Covariant)**: 사상 방향 보존. `fmap`. 반환 타입 위치
- **반변(Contravariant)**: 사상 방향 뒤집힘. `contramap`. 인자 타입 위치
- **프로펑터(Profunctor)**: 첫 인자 반변 + 둘째 인자 공변. `dimap f g h = g . h . f`
- 함수 타입 `a → b`는 a에 대해 반변, b에 대해 공변인 프로펑터
