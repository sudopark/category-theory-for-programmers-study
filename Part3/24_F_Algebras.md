# Chapter 24: F-Algebras — F-대수

> **핵심 개념**: **F-대수(F-Algebra)**는 펑터 F를 사용하여 대수적 구조를 일반화한다. 대수는 "수식을 평가하는 것"이고, F-대수는 F가 정의하는 수식의 구조를 대상 a로 평가하는 사상 `F a → a`이다. **시작 대수(initial algebra)**는 재귀적 데이터 타입에 대응하며, **카타모피즘(catamorphism)**은 이 구조에 대한 일반적인 접기(fold)다.

---

## 대수의 일반화

전통적인 대수 구조를 떠올려보자:

- **모노이드**: 집합 m, 이항 연산 `m × m → m`, 단위 원소 `() → m`
- **군(Group)**: 모노이드 + 역원 `m → m`

이 모든 연산을 **하나의 펑터**로 묶을 수 있다:

```haskell
-- 모노이드의 연산을 하나로
data MonF a = MEmpty | MAppend a a

-- 이항 연산과 단위 원소를 합 타입으로 합침:
-- MonF a = () + a × a
-- 즉, MonF a ≅ Either () (a, a)
```

---

## F-대수의 정의

**F-대수**는 다음의 쌍이다:

- **운반 대상(carrier)** a: 카테고리의 대상
- **구조 사상(structure map)** `alg :: F a → a`: 펑터 F가 정의하는 "수식"을 a에서 "평가"

```haskell
type Algebra f a = f a -> a
```

### 예시: 수식 평가

```haskell
-- 수식의 구조를 정의하는 펑터
data ExprF a = Const Int    -- 상수
             | Add a a      -- 덧셈
             | Mul a a      -- 곱셈

-- Int를 운반 대상으로 하는 F-대수 (수식 평가기)
evalAlg :: Algebra ExprF Int
evalAlg (Const n)   = n
evalAlg (Add x y)   = x + y
evalAlg (Mul x y)   = x * y

-- String을 운반 대상으로 하는 F-대수 (수식 출력기)
showAlg :: Algebra ExprF String
showAlg (Const n)   = show n
showAlg (Add x y)   = "(" ++ x ++ " + " ++ y ++ ")"
showAlg (Mul x y)   = "(" ++ x ++ " * " ++ y ++ ")"
```

같은 펑터 F에 대해 **운반 대상과 구조 사상을 바꾸면** 다른 해석(평가, 출력, 최적화 등)을 얻는다.

---

## F-대수의 카테고리

F-대수들은 카테고리를 이룬다:

- **대상**: F-대수 (a, alg :: F a → a)
- **사상(대수 준동형)**: 함수 `m :: a → b`로, 다음 다이어그램이 가환:

```
F a ---F m--→ F b
 |             |
 alg           alg'
 |             |
 ↓             ↓
 a ----m----→ b
```

즉: `m . alg = alg' . F m`

---

## 시작 대수와 재귀적 타입

F-대수의 카테고리에서 **시작 대상(initial object)**이 존재하면, 이것이 **시작 대수(initial algebra)**다.

**Lambek 정리**: 시작 대수의 구조 사상 `i :: F(Fix F) → Fix F`는 **동형사상**이다.

```
F(Fix F) ≅ Fix F
```

이것은 바로 **재귀적 타입의 정의**다!

```haskell
-- 부동점 타입
newtype Fix f = Fix (f (Fix f))

unfix :: Fix f -> f (Fix f)
unfix (Fix x) = x
```

### 예시: 리스트

```haskell
data ListF a b = NilF | ConsF a b

-- Fix (ListF a) ≅ ListF a (Fix (ListF a))
--               ≅ NilF | ConsF a (Fix (ListF a))
-- 이것이 바로 List a의 정의!
```

---

## 카타모피즘 (Catamorphism)

시작 대수에서 임의의 F-대수로의 **유일한 준동형**이 **카타모피즘(catamorphism)**이다. 이것은 재귀적 데이터 타입에 대한 **일반화된 fold**다.

```haskell
cata :: Functor f => Algebra f a -> Fix f -> a
cata alg = alg . fmap (cata alg) . unfix
```

동작 과정:
1. `unfix`: 재귀 구조를 한 단계 펼침 (`Fix f → f (Fix f)`)
2. `fmap (cata alg)`: 하위 구조에 재귀적으로 카타모피즘 적용
3. `alg`: 결과를 평가

```haskell
-- 리스트의 카타모피즘 = foldr
-- 자연수의 카타모피즘 = 원시 재귀
-- 트리의 카타모피즘 = 트리 접기
```

---

## F-쌍대대수와 아나모피즘

쌍대를 취하면:

- **F-쌍대대수(F-Coalgebra)**: `a → F a` — 상태에서 관찰을 생성
- **끝 쌍대대수(terminal coalgebra)**: 무한 구조 (무한 스트림 등)
- **아나모피즘(anamorphism)**: 일반화된 unfold

```haskell
type Coalgebra f a = a -> f a

ana :: Functor f => Coalgebra f a -> a -> Fix f
ana coalg = Fix . fmap (ana coalg) . coalg
```

---

## 정리

- **F-대수**: 펑터 F + 운반 대상 a + 구조 사상 `F a → a`
- **펑터 F**: 수식의 "형태"를 정의. 운반 대상이 "의미"를 부여
- **시작 대수**: F-대수 카테고리의 시작 대상. `Fix F ≅ F(Fix F)` (Lambek 정리)
- **Fix**: 펑터의 부동점. 재귀적 데이터 타입에 대응
- **카타모피즘** `cata`: 시작 대수에서의 유일한 준동형. 일반화된 fold
- **F-쌍대대수**: `a → F a`. 상태 기반 시스템, 관찰 가능 행동
- **아나모피즘** `ana`: 끝 쌍대대수로의 유일한 준동형. 일반화된 unfold
