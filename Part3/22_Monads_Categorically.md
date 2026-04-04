# Chapter 22: Monads Categorically — 모나드의 범주적 정의

> **핵심 개념**: 프로그래머에게 모나드는 bind와 return이지만, 수학자에게 모나드는 자기 함자(endofunctor) T와 두 자연 변환 **η(단위)**와 **μ(곱)**으로 이루어진 대수적 구조다. μ는 `join`에, η는 `return`에 대응하며, 모나드 법칙은 모노이드 법칙의 범주적 일반화다.

---

## 모나드의 범주적 정의

**모나드**는 다음 세 가지로 구성된다:

1. **자기 함자** T: C → C
2. **단위(unit)** η: Id → T (자연 변환)
3. **곱(multiplication)** μ: T² → T (자연 변환), 여기서 T² = T ∘ T

```haskell
-- Haskell 대응
T     ↔ 타입 생성자 m
η_a   ↔ return :: a -> m a
μ_a   ↔ join :: m (m a) -> m a
```

---

## 모나드 법칙

### 결합법칙

```
μ ∘ Tμ = μ ∘ μT
```

다이어그램:
```
T³ ---Tμ--→ T²
 |            |
 μT           μ
 |            |
 ↓            ↓
T² ----μ---→ T
```

Haskell로: `join . fmap join = join . join`

직관: 삼중으로 감싸진 `m(m(m a))`를 풀 때, 바깥부터 풀든(join . join) 안쪽부터 풀든(join . fmap join) 결과가 같다.

### 단위 법칙

```
μ ∘ ηT = id = μ ∘ Tη
```

다이어그램:
```
     ηT          Tη
T ---→ T² ←--- T
 \      |      /
  \     μ     /
   \    |    /
    \   ↓   /
      → T ←
       id
```

Haskell로:
```haskell
join . return      = id    -- 바깥에 return 후 join
join . fmap return = id    -- 안쪽에 return 후 join
```

---

## 모나드 = 자기 함자 카테고리의 모노이드

매우 유명한 문구: **"모나드는 자기 함자 카테고리의 모노이드일 뿐이다."**

이것을 풀어보면:

- **자기 함자 카테고리** [C, C]: 대상 = C에서 C로의 펑터, 사상 = 자연 변환
- 이 카테고리에서 "곱"은 **펑터 합성**: T ∘ T = T²
- **모노이드**: 이항 연산(μ: T² → T)과 단위 원소(η: Id → T)를 갖추고 결합법칙과 단위법칙을 만족

집합의 모노이드에서:
- 이항 연산: `m × m → m` (mappend)
- 단위 원소: `() → m` (mempty)

자기 함자 카테고리의 모노이드에서:
- 이항 연산: `T ∘ T → T` (μ = join)
- 단위 원소: `Id → T` (η = return)

구조가 완전히 대응된다!

---

## 수반에서 모나드 유도

수반 L ⊣ R (L: C → D, R: D → C)에서:

- T = R ∘ L (자기 함자)
- η = 수반의 단위 (Id → R∘L)
- μ는 쌍대단위 ε를 사용: μ = R ε L :: R∘L∘R∘L → R∘L

```
μ = RεL : RLRL → RL
```

삼각 항등식이 모나드 법칙을 보장한다.

---

## 자연 변환의 합성과 모나드

모나드 법칙에서 `Tμ`와 `μT`는 **수평 합성(horizontal composition)**이다:

- `Tμ`: 펑터 T와 자연 변환 μ의 수평 합성 (위스커링)
- `μT`: 자연 변환 μ와 펑터 T의 수평 합성

이것은 2-카테고리 **Cat**에서의 구조이며, 모나드는 2-카테고리에서의 모노이드 대상이다.

---

## Haskell의 Monad 타입클래스

```haskell
class Functor m => Monad m where
    return :: a -> m a           -- η
    (>>=)  :: m a -> (a -> m b) -> m b  -- bind (join + fmap에서 유도)
    
-- join은 bind에서 유도 가능:
join :: Monad m => m (m a) -> m a
join mma = mma >>= id

-- bind는 join과 fmap에서 유도 가능:
ma >>= f = join (fmap f ma)
```

---

## 정리

- **범주적 모나드**: 자기 함자 T + 단위 η: Id→T + 곱 μ: T²→T
- **η = return**, **μ = join**
- **모나드 법칙**: 결합법칙 `μ∘Tμ = μ∘μT`, 단위법칙 `μ∘ηT = id = μ∘Tη`
- **"모나드 = 자기 함자 카테고리의 모노이드"**: μ는 mappend, η는 mempty에 대응
- **수반에서 모나드**: T = R∘L, η = 수반의 단위, μ = RεL
- bind `>>=`와 join은 상호 정의 가능: `ma >>= f = join (fmap f ma)`
