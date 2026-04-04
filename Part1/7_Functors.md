# Chapter 7: Functors — 펑터

> **핵심 개념**: **펑터(Functor)**는 카테고리 사이의 구조를 보존하는 매핑이다. 대상을 대상으로, 사상을 사상으로 매핑하되, 합성과 항등 사상을 보존한다. 프로그래밍에서 펑터는 **타입 생성자 + fmap**으로 구현되며, Maybe, List, Reader 등 다양한 예시가 있다.

---

## 펑터의 정의

두 카테고리 **C**와 **D**가 있을 때, 펑터 F: C → D는 다음을 만족하는 매핑이다:

1. **대상의 매핑**: C의 대상 a를 D의 대상 Fa로 보낸다
2. **사상의 매핑**: C의 사상 `f :: a → b`를 D의 사상 `Ff :: Fa → Fb`로 보낸다
3. **합성 보존**: `F(g ∘ f) = Fg ∘ Ff`
4. **항등 보존**: `F(id_a) = id_{Fa}`

```haskell
-- 대상 매핑
a       ↦ Fa
-- 사상 매핑
f :: a -> b
Ff :: Fa -> Fb
-- 합성 보존
F(g . f) = Fg . Ff
-- 항등 보존
F(id_a) = id_(Fa)
```

직관적으로, 펑터는 카테고리의 **구조를 보존하면서** 다른 카테고리로 옮기는 것이다. 원래 카테고리에서 연결된 것은 옮긴 후에도 여전히 연결되어 있다. 미적분에서의 연속 함수가 "찢어지지 않는" 함수인 것처럼, 펑터도 카테고리의 구조를 "찢지 않는다."

### 특수한 펑터들

- **상수 펑터(Constant Functor)** Δ_c: 모든 대상을 하나의 대상 c로, 모든 사상을 id_c로 매핑. 블랙홀처럼 모든 것을 하나로 압축
- **자기 함자(Endofunctor)**: 한 카테고리를 자기 자신으로 매핑하는 펑터. 프로그래밍에서 다루는 펑터는 대부분 이것
- **싱글톤 카테고리에서의 펑터**: 대상 카테고리에서 하나의 대상을 "선택"하는 역할

---

## 프로그래밍에서의 펑터

타입과 함수의 카테고리에서 자기 함자(endofunctor)는:
- **대상의 매핑** = 타입 생성자 (예: `Maybe`, `[]`, `(->) r`)
- **사상의 매핑** = `fmap` 함수

### Maybe 펑터

```haskell
data Maybe a = Nothing | Just a

-- fmap: 함수를 Maybe 안의 값에 적용
fmap :: (a -> b) -> Maybe a -> Maybe b
fmap _ Nothing  = Nothing
fmap f (Just x) = Just (f x)
```

`fmap`은 함수를 **리프팅(lifting)**한다고도 말한다: 일반 함수 `a → b`를 Maybe 세계의 함수 `Maybe a → Maybe b`로 끌어올린다.

```swift
enum Optional<T> {
    case nothing
    case just(T)
    
    func fmap<B>(_ f: (T) -> B) -> Optional<B> {
        switch self {
        case .nothing: return .nothing
        case .just(let x): return .just(f(x))
        }
    }
}
```

### 펑터 법칙의 증명 (Equational Reasoning)

Haskell의 함수는 등식으로 정의되므로, 양변을 치환하여 증명할 수 있다:

**항등 법칙** `fmap id = id`:

```haskell
-- Nothing 케이스
fmap id Nothing
  = Nothing           -- fmap 정의
  = id Nothing        -- id 정의

-- Just 케이스
fmap id (Just x)
  = Just (id x)       -- fmap 정의
  = Just x            -- id 정의
  = id (Just x)       -- id 정의
```

**합성 법칙** `fmap (g . f) = fmap g . fmap f`:

```haskell
-- Nothing 케이스
fmap (g . f) Nothing = Nothing = fmap g (fmap f Nothing)

-- Just 케이스
fmap (g . f) (Just x)
  = Just ((g . f) x)           -- fmap 정의
  = Just (g (f x))             -- 합성 정의
  = fmap g (Just (f x))        -- fmap 정의 역적용
  = fmap g (fmap f (Just x))   -- fmap 정의 역적용
  = (fmap g . fmap f) (Just x) -- 합성 정의
```

---

### List 펑터

```haskell
data List a = Nil | Cons a (List a)

instance Functor List where
    fmap _ Nil        = Nil
    fmap f (Cons x t) = Cons (f x) (fmap f t)
```

`fmap f`는 리스트의 각 원소에 `f`를 적용한다. 재귀적으로 정의되며, Nil에 도달하면 종료된다.

### Reader 펑터

타입 `a`를 `r → a` (r에서 a로의 함수)로 매핑하는 펑터. 타입 생성자는 `(->) r`이다.

```haskell
fmap :: (a -> b) -> (r -> a) -> (r -> b)
```

`a → b` 함수와 `r → a` 함수가 있으면 이 둘을 합성하여 `r → b`를 얻는다:

```haskell
instance Functor ((->) r) where
    fmap f g = f . g
    -- 즉, fmap = (.)  (함수 합성 그 자체!)
```

> Reader 펑터에서 fmap은 곧 **함수 합성**이다. 이는 펑터가 단순한 "컨테이너"를 넘어서는 개념임을 보여준다.

---

## 펑터와 컨테이너

Maybe나 List처럼 값을 담고 있는 컨테이너는 펑터의 직관적 예시다. 하지만 Reader 펑터처럼 함수 자체가 펑터가 될 수도 있다.

Haskell에서는 게으른 평가(lazy evaluation) 덕분에 데이터와 코드의 경계가 모호하다:

```haskell
nats :: [Integer]
nats = [1..]  -- 무한한 자연수 리스트 (실제로는 요청 시 생성하는 함수)
```

펑터의 핵심은 **값에 접근하는 것이 아니라, 값을 변환하는 것**이다. fmap은:
- 값이 있으면 변환 후 접근 가능
- 값이 없어도 변환은 올바르게 합성되어야 함
- 항등 변환은 아무 변화도 일으키지 않아야 함

### Const 펑터

극단적인 예시: 타입 인자를 완전히 무시하는 펑터.

```haskell
data Const c a = Const c

instance Functor (Const c) where
    fmap _ (Const v) = Const v
```

`Const c a`는 c 타입의 값만 저장하고 a는 **유령 타입(phantom type)**이다. fmap은 함수 인자를 무시한다. 이것은 상수 펑터 Δ_c의 Haskell 구현이며, 나중에 Yoneda 보조정리 등에서 중요한 역할을 한다.

---

## 펑터의 합성

두 펑터를 합성하면 새로운 펑터가 된다. 대상 매핑과 사상 매핑을 각각 합성하면 되고, 합성/항등 보존도 자동으로 만족된다.

```haskell
-- Maybe와 List 펑터의 합성: Maybe [a]
mis :: Maybe [Int]
mis = Just [1, 2, 3]

-- 내부의 각 정수를 제곱하려면 fmap을 두 번 적용
mis2 = fmap (fmap square) mis
-- = Just [1, 4, 9]

-- 이것은 fmap의 합성과 같다
mis2 = (fmap . fmap) square mis
```

바깥쪽 `fmap`은 Maybe 인스턴스를, 안쪽 `fmap`은 List 인스턴스를 사용한다.

### Functor 타입 클래스

Haskell은 **타입 클래스(typeclass)**로 펑터 인터페이스를 추상화한다:

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

여기서 `f`는 타입이 아니라 **타입 생성자**다 (종류가 `* -> *`). 구체적인 펑터는 인스턴스를 선언하여 fmap을 구현한다:

```haskell
instance Functor Maybe where
    fmap _ Nothing  = Nothing
    fmap f (Just x) = Just (f x)
```

### 카테고리들의 카테고리 (Cat)

펑터는 합성이 가능하고 결합법칙을 만족하며, 항등 펑터도 존재한다. 따라서 **대상이 카테고리이고 사상이 펑터인 카테고리**를 생각할 수 있다. 이를 **Cat**(작은 카테고리들의 카테고리)이라 부른다.

---

## 정리

- **펑터**: 카테고리 간의 구조 보존 매핑 (대상→대상, 사상→사상, 합성/항등 보존)
- **프로그래밍에서**: 타입 생성자(대상 매핑) + fmap(사상 매핑)
- **펑터 법칙**: `fmap id = id`, `fmap (g . f) = fmap g . fmap f`
- **Maybe 펑터**: Nothing은 그대로, Just 안의 값에 함수 적용
- **List 펑터**: 각 원소에 함수를 적용 (map)
- **Reader 펑터** `(->) r`: fmap = 함수 합성 `(.)`
- **Const 펑터**: 타입 인자를 무시하는 극단적 펑터
- **펑터 합성**: `(fmap . fmap)` — 중첩된 펑터의 내부까지 함수를 전달
- **Cat**: 카테고리들의 카테고리. 대상=카테고리, 사상=펑터
