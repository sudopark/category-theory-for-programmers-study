# Chapter 27: Kan Extensions — 칸 확장

> **핵심 개념**: **칸 확장(Kan Extension)**은 "카테고리 이론의 모든 개념은 칸 확장의 특수한 경우"라 할 만큼 핵심적인 개념이다(Saunders Mac Lane). 펑터 K: A → C를 따라 펑터 D: A → B를 확장하는 가장 좋은 방법을 제공하며, 극한, 쌍대극한, 수반, 요네다 보조정리 등이 모두 칸 확장의 특수 경우다.

---

## 동기: 펑터의 확장

펑터 D: A → B와 K: A → C가 주어졌을 때, C에서 B로의 펑터 F를 "D와 호환되게" 만들고 싶다. 즉, F ∘ K ≈ D가 되는 F를 찾고 싶다.

```
      K
A --------→ C
 \          /
  D   ⇓η   F    (우칸 확장)
   \      /
    ↓   ↓
      B
```

---

## 우칸 확장 (Right Kan Extension)

펑터 K: A → C와 D: A → B에 대한 **우칸 확장** Ran_K D: C → B는:

> 자연 변환 ε: (Ran_K D) ∘ K → D가 존재하여, 임의의 펑터 F: C → B와 자연 변환 σ: F∘K → D에 대해, **유일한** 자연 변환 τ: F → Ran_K D가 존재하여 σ = ε ∘ (τ ∘ K)

끝(end)을 사용한 공식:

```
(Ran_K D)(c) = ∫_a B(K a, c) ⇒ D a
```

여기서 ⇒는 지수(함수 객체)를 뜻한다.

### Haskell에서

```haskell
newtype Ran k d c = Ran (forall a. (a -> c) -> d a)
-- 또는 더 정확히: Ran k d c = forall a. (k a -> c) -> d a
-- 이것은 끝(forall)으로 표현된 우칸 확장
```

---

## 좌칸 확장 (Left Kan Extension)

쌍대:

```
      K
A --------→ C
 \          /
  D   ⇑η   F    (좌칸 확장)
   \      /
    ↓   ↓
      B
```

쌍대끝(coend) 공식:

```
(Lan_K D)(c) = ∫^a C(K a, c) × D a
```

### Haskell에서

```haskell
data Lan k d c where
    Lan :: (k a -> c) -> d a -> Lan k d c
-- 존재 타입(exists a)으로 표현된 좌칸 확장
```

---

## 칸 확장의 특수 경우들

### 극한과 쌍대극한

K: A → 1 (끝 대상으로의 유일한 펑터)일 때:

```
Ran_K D = Lim D   (극한)
Lan_K D = Colim D  (쌍대극한)
```

### 수반

L ⊣ R일 때, R은 L을 따른 항등 펑터의 우칸 확장이다:

```
R ≅ Ran_L Id
```

### 요네다 보조정리

K = Id일 때:

```
Ran_Id D = D   (항등 펑터를 따른 칸 확장은 자기 자신)
```

이것이 요네다 보조정리의 범주적 표현이다.

---

## 프로그래밍에서의 칸 확장

칸 확장은 프로그래밍에서도 실용적이다:

```haskell
-- 우칸 확장: "Density 코모나드" 등에 활용
-- 좌칸 확장: "Day convolution" 등에 활용

-- 자유 펑터로서의 좌칸 확장
-- Lan_Id F ≅ F  (요네다 보조정리)

-- 타입 수준의 좌칸 확장
type family Lan (k :: i -> j) (d :: i -> Type) (c :: j) where ...
```

---

## 정리

- **칸 확장**: 펑터 K를 따라 펑터 D를 확장하는 보편적 방법
- **우칸 확장** Ran_K D: 끝으로 표현. `(Ran_K D)(c) = ∫_a [K a, c] ⇒ D a`
- **좌칸 확장** Lan_K D: 쌍대끝으로 표현. `(Lan_K D)(c) = ∫^a C(K a, c) × D a`
- **Haskell**: Ran은 `forall a. (k a -> c) -> d a`, Lan은 `exists a. (k a -> c, d a)`
- **특수 경우**: 극한/쌍대극한 (K → 1), 수반 (R ≅ Ran_L Id), 요네다 (K = Id)
- "카테고리 이론의 모든 개념은 칸 확장의 특수 경우" — Mac Lane
