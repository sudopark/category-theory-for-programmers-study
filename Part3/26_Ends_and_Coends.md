# Chapter 26: Ends and Coends — 끝과 쌍대끝

> **핵심 개념**: **끝(End)**과 **쌍대끝(Coend)**은 프로펑터(두 인자 펑터)에 대한 일반화된 극한과 쌍대극한이다. 끝은 자연 변환의 집합을 일반화하고, 쌍대끝은 존재 타입(existential type)을 일반화한다. 이들은 요네다 보조정리, 칸 확장 등 고급 구성의 핵심 도구다.

---

## 동기: 자연 변환을 일반화하기

두 펑터 F, G: C → Set 사이의 자연 변환의 집합:

```
Nat(F, G) = { α | ∀a. α_a :: F a → G a, 자연성 만족 }
```

이것을 "모든 a에 대한 곱(product)에서 자연성 조건으로 제한한 것"으로 볼 수 있다. **끝**은 이 아이디어를 프로펑터로 일반화한다.

---

## 끝 (End)

프로펑터 P: C^op × C → Set에 대한 **끝(end)**:

```
∫_c P(c, c) = { 자연적으로 호환되는 P(c,c)의 원소들의 모임 }
```

기호 ∫는 적분 기호를 빌려 쓴다. 첨자 c는 "c에 대해 끝을 취한다"는 뜻이다.

### 쐐기 조건 (Wedge Condition)

끝의 원소는 **쐐기(wedge)** 조건을 만족해야 한다. 임의의 사상 `f :: a → b`에 대해:

```
P(f, id_b) ∘ π_b = P(id_a, f) ∘ π_a
```

여기서 π_c는 끝에서 P(c,c)로의 투영이다.

### 자연 변환 = 끝

두 펑터 F, G에 대해 프로펑터 P(a, b) = Set(F a, G b)를 정의하면:

```
∫_c Set(F c, G c) ≅ Nat(F, G)
```

**자연 변환의 집합은 이 프로펑터의 끝이다!**

### Haskell에서

Haskell의 `forall`이 끝에 해당한다:

```haskell
-- 끝 = forall (보편 양화)
-- ∫_a P(a, a) ≅ forall a. P a a

-- 자연 변환의 예:
type Nat f g = forall a. f a -> g a
-- 이것은 프로펑터 (a, b) ↦ (f a -> g b)의 끝
```

---

## 쌍대끝 (Coend)

끝의 쌍대. 프로펑터 P: C^op × C → Set에 대한 **쌍대끝(coend)**:

```
∫^c P(c, c) = ⨿_c P(c,c) / ~ (동치 관계로 쿼우션트)
```

모든 P(c,c)의 분리 합집합에서 **쌍대쐐기 조건**에 의한 동치 관계로 식별한 것이다.

### Haskell에서

쌍대끝은 **존재 타입(existential type)**에 해당한다:

```haskell
-- 쌍대끝 = exists (존재 양화)
-- ∫^a P(a, a) ≅ exists a. P a a

data Coend p where
    Coend :: p a a -> Coend p   -- 어떤 a가 존재하여 p a a의 값
```

---

## 요네다 보조정리의 끝/쌍대끝 표현

```
-- 요네다 보조정리 (끝 형태)
∫_x Set(C(a, x), F x) ≅ F a

-- Haskell:
-- forall x. (a -> x) -> f x  ≅  f a

-- 쌍대 요네다 (쌍대끝 형태)
∫^x C(x, a) × F x ≅ F a

-- Haskell:
-- exists x. (x -> a, f x)  ≅  f a  (fmap으로 증명)
```

---

## 닌자 요네다 보조정리

쌍대끝과 끝을 이용한 강력한 변환:

```haskell
-- 모든 펑터 f에 대해:
-- f a ≅ ∫^x C(a, x) × F x    (쌍대 요네다)
-- f a ≅ ∫_x C(x, a) → F x    (요네다)
```

이들은 최적화(fusion)와 자유 정리의 기반이 된다.

---

## 정리

- **끝(End)** `∫_c P(c,c)`: 프로펑터의 일반화된 극한. 쐐기 조건을 만족하는 원소들의 집합
- **쌍대끝(Coend)** `∫^c P(c,c)`: 프로펑터의 일반화된 쌍대극한. 분리 합집합의 쿼우션트
- **Haskell**: 끝 = `forall` (보편 양화), 쌍대끝 = `exists` (존재 양화)
- **자연 변환 = 끝**: `Nat(F,G) ≅ ∫_c Set(Fc, Gc)`
- **요네다 보조정리**: 끝과 쌍대끝으로 표현 가능
- 끝/쌍대끝은 칸 확장, 프로펑터 광학(optics) 등 고급 주제의 핵심 도구
