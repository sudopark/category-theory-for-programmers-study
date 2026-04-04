# Chapter 14: Representable Functors — 표현 가능 펑터

> **핵심 개념**: **홈 펑터(hom-functor)** `C(a, -)`는 대상 a를 고정하고 다른 대상 x에 대한 사상의 집합을 매핑하는 펑터다. 어떤 펑터 F가 홈 펑터와 **자연 동형(naturally isomorphic)**이면, F를 **표현 가능 펑터(representable functor)**라 하고 a를 **표현 대상(representing object)**이라 한다. 이 개념은 카테고리의 대상을 집합(함수)으로 번역하는 강력한 도구다.

---

## 홈 펑터 (Hom-Functor)

### 공변 홈 펑터: C(a, -)

대상 a를 고정하면, `x ↦ C(a, x)` (a에서 x로의 모든 사상의 집합)은 **C에서 Set으로의 펑터**가 된다:

- **대상 매핑**: x ↦ C(a, x) (홈셋)
- **사상 매핑**: 사상 `f :: x → y`에 대해, `C(a, f) :: C(a,x) → C(a,y)`는 **후합성(postcomposition)** — `h ↦ f ∘ h`

```haskell
-- Haskell에서 홈 펑터는 Reader 펑터
-- C(a, -) = (->) a = Reader a

type Reader a x = a -> x

instance Functor (Reader a) where
    fmap f g = f . g   -- 후합성
```

### 반변 홈 펑터: C(-, a)

대상 a를 두 번째 인자에 고정하면, `x ↦ C(x, a)`는 **반변 펑터**가 된다:

- **사상 매핑**: `f :: x → y`에 대해, `C(f, a) :: C(y,a) → C(x,a)`는 **전합성(precomposition)** — `h ↦ h ∘ f`

```haskell
-- 반변 홈 펑터
-- C(-, a) = Op a

newtype Op a x = Op (x -> a)

instance Contravariant (Op a) where
    contramap f (Op g) = Op (g . f)   -- 전합성
```

### 홈 쌍펑터: C(-, -)

두 인자 모두 변하게 하면, `C(-, -)` 는 C^op × C → Set인 **프로펑터**다:

```haskell
-- 프로펑터로서의 함수 타입
instance Profunctor (->) where
    dimap f g h = g . h . f
```

---

## 표현 가능 펑터 (Representable Functor)

펑터 F: C → Set이 **표현 가능(representable)**하다는 것은, 어떤 대상 a가 존재하여:

```
F ≅ C(a, -)    -- 자연 동형
```

이 a를 **표현 대상(representing object)**이라 한다. 자연 동형이란 각 대상 x에서 `F(x) ≅ C(a, x)`이고, 이 동형이 x에 대해 자연적(natural)이라는 뜻이다.

### 직관

표현 가능 펑터는 "함수로 구현할 수 있는" 펑터다. `F(x)`의 각 원소가 `a → x` 함수 하나에 대응된다면, F는 표현 가능하다.

### 예시: Stream 펑터

무한 스트림은 자연수를 인덱스로 사용하는 것과 같다:

```haskell
data Stream a = Cons a (Stream a)

-- Stream a ≅ (Integer -> a) ≅ C(Integer, a)
-- Stream은 Integer로 표현 가능!

-- 인덱스로 접근
nth :: Integer -> Stream a -> a
nth 0 (Cons x _)  = x
nth n (Cons _ xs) = nth (n-1) xs

-- 함수에서 스트림으로
tabulate :: (Integer -> a) -> Stream a
tabulate f = Cons (f 0) (tabulate (f . (+1)))
```

### 예시: Reader 펑터

`Reader a`는 자명하게 표현 가능하다 — 그 자체가 홈 펑터 C(a, -)이므로.

### 비예시: List 펑터

리스트는 표현 가능하지 **않다**. `[a]`의 원소 개수가 정해져 있지 않으므로 하나의 함수 타입으로 동형을 만들 수 없다.

---

## Haskell에서의 표현 가능 펑터

```haskell
class Representable f where
    type Rep f :: *                    -- 표현 대상의 타입
    tabulate :: (Rep f -> a) -> f a    -- 함수에서 펑터 값으로
    index    :: f a -> Rep f -> a      -- 펑터 값에서 함수로

-- tabulate과 index는 역함수 관계:
-- tabulate . index = id
-- index . tabulate = id
```

```haskell
-- Stream의 표현
instance Representable Stream where
    type Rep Stream = Integer
    tabulate f = Cons (f 0) (tabulate (f . (+1)))
    index (Cons x _)  0 = x
    index (Cons _ xs) n = index xs (n-1)
```

---

## 표현 가능 펑터의 성질

표현 가능 펑터는 매우 좋은 성질을 가진다:

1. **자동으로 fmap을 구현할 수 있다**: `fmap f = tabulate . (f .) . index`
2. **극한을 보존한다**: 표현 가능 펑터는 모든 극한을 보존
3. **메모이제이션**: `tabulate`은 함수를 데이터 구조로 메모이제이션, `index`는 룩업

---

## 정리

- **홈 펑터** `C(a, -)`: 대상 a를 고정하고 홈셋을 매핑하는 공변 펑터. Haskell에서 `Reader a`
- **반변 홈 펑터** `C(-, a)`: 반변 버전. Haskell에서 `Op a`
- **표현 가능 펑터**: 홈 펑터와 자연 동형인 펑터. `F ≅ C(a, -)`
- **tabulate/index**: 함수 ↔ 데이터 구조 사이의 동형. 메모이제이션과 룩업
- **Stream**: 자연수로 표현 가능한 펑터의 대표적 예시
- List는 표현 가능하지 않음 (길이가 가변적)
- 표현 가능 펑터는 극한을 보존하는 등 좋은 성질을 가짐
