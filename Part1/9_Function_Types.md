# Chapter 9: Function Types — 함수 타입

> **핵심 개념**: 함수 타입은 카테고리 내부에서 **지수 객체(exponential)**로 정의된다. 보편적 구성으로 함수 객체를 정의하고, 커링을 통해 곱과 지수의 관계를 이해한다. 이 구조를 갖춘 카테고리가 **데카르트 닫힌 카테고리(CCC)**이며, **Curry-Howard 동형**은 타입-논리 대응을 완성한다.

---

## 함수 객체와 내부 홈 (Internal Hom)

정수나 Bool은 단순한 값들의 집합이지만, 함수 타입 `a → b`는 특별하다. 이것은 대상 a에서 b로의 **사상들의 집합(hom-set)** `C(a, b)`을 의미한다.

**Set** 카테고리에서는 hom-set 자체가 집합이므로 같은 카테고리의 대상이 된다. 이런 자기 참조적 성질 때문에 함수 타입을 다른 타입처럼 다룰 수 있다. 이것을 **내부 홈(internal hom)**이라 한다.

일반 카테고리에서는 hom-set이 반드시 같은 카테고리의 대상이 되지 않는다. 그래서 함수 객체를 **보편적 구성**으로 정의한다.

---

## 함수 객체의 보편적 구성

함수 객체 `a ⇒ b`를 정의하기 위해 **평가(evaluation)** 사상을 핵심 패턴으로 사용한다:

```
eval :: (a ⇒ b) × a → b
```

이것은 "함수와 인자의 쌍을 받아 결과를 돌려준다"는 뜻이다.

**보편적 성질**: 임의의 대상 z와 사상 `g :: z × a → b`에 대해, 유일한 사상 `h :: z → (a ⇒ b)`가 존재하여:

```
g = eval ∘ (h × id_a)
```

즉, `g`를 `h`로 분해(factorize)할 수 있다. 이 `h`가 `g`의 **커링(currying)**이다.

```haskell
-- Haskell에서 함수 객체는 그냥 함수 타입 a -> b
eval :: (a -> b, a) -> b
eval (f, x) = f x
```

---

## 커링과 언커링

**커링(Currying)**: 두 인자를 받는 함수를 "함수를 반환하는 함수"로 변환

```haskell
curry :: ((a, b) -> c) -> (a -> (b -> c))
curry f a b = f (a, b)

uncurry :: (a -> (b -> c)) -> ((a, b) -> c)
uncurry f (a, b) = f a b
```

```swift
func curry<A, B, C>(_ f: @escaping (A, B) -> C) -> (A) -> (B) -> C {
    return { a in { b in f(a, b) } }
}

func uncurry<A, B, C>(_ f: @escaping (A) -> (B) -> C) -> (A, B) -> C {
    return { pair in f(pair.0)(pair.1) }
}
```

Haskell에서 모든 함수는 기본적으로 커링되어 있다:

```haskell
-- 이 두 표현은 동일하다
f :: a -> b -> c
f :: a -> (b -> c)
```

커링은 보편적 구성에서의 factorizer에 해당한다. `z × a → b`를 `z → (a ⇒ b)`로 변환하는 것이 바로 curry다.

---

## 지수 객체 (Exponentials)

함수 객체 `a ⇒ b`를 **지수(exponential)** 표기법으로 **b^a**라 쓴다.

이 표기가 자연스러운 이유는 집합의 크기(원소 수)를 생각하면 된다:
- `|a → b| = |b|^|a|` — a에서 b로의 함수의 수는 b의 원소 수의 a의 원소 수 제곱

예시:
- `Bool → Bool`: 2^2 = 4가지 함수 (id, not, const True, const False)
- `() → a`: a^1 = a — a의 원소를 하나 고르는 것 (a와 동형)
- `Void → a`: a^0 = 1 — absurd 함수 하나뿐 (Unit과 동형)

---

## 데카르트 닫힌 카테고리 (Cartesian Closed Category, CCC)

다음 세 가지를 갖춘 카테고리를 **데카르트 닫힌 카테고리(CCC)**라 한다:

1. **끝 대상(terminal object)**: Unit에 해당
2. **임의의 두 대상의 곱**: 페어 (a, b)
3. **임의의 두 대상의 지수**: 함수 객체 a → b

**Set**은 CCC의 대표적 예시이며, 타입이 있는 람다 계산법(typed lambda calculus)의 내부 언어에 해당한다.

---

## 지수와 대수적 데이터 타입

지수 표기법을 쓰면 타입의 대수가 수의 대수와 완벽히 대응된다:

| 수의 법칙 | 타입의 대응 |
|-----------|-----------|
| a^0 = 1 | `Void → a ≅ ()` |
| 1^a = 1 | `a → () ≅ ()` |
| a^1 = a | `() → a ≅ a` |
| a^(b+c) = a^b × a^c | `Either b c → a ≅ (b → a, c → a)` |
| (a^b)^c = a^(b×c) | `c → (b → a) ≅ (b, c) → a` (커링!) |
| (a × b)^c = a^c × b^c | `c → (a, b) ≅ (c → a, c → b)` |

특히 `(a^b)^c = a^(b×c)`는 **커링**의 수학적 표현이다!

그리고 `a^(b+c) = a^b × a^c`는 실용적으로:

```haskell
-- Either b c를 받는 함수를 정의하려면, b를 받는 함수와 c를 받는 함수 두 개가 필요하다
f :: Either b c -> a
-- 이것은 (b -> a, c -> a) 쌍과 동형
```

---

## Curry-Howard 동형

**Curry-Howard 동형(대응)**은 타입과 논리적 명제 사이의 깊은 연결이다:

| 논리 | 타입 |
|------|------|
| False | `Void` |
| True | `()` |
| a ∧ b (AND) | `(a, b)` |
| a ∨ b (OR) | `Either a b` |
| a ⇒ b (함의, implication) | `a → b` |

**"타입은 명제이고, 프로그램은 증명이다."**

- 함수 `a → b`가 존재한다 = "a이면 b이다"라는 명제의 증명이 존재한다
- `absurd :: Void → a` = 거짓에서는 무엇이든 따라 나온다 (ex falso quodlibet)
- `unit :: a → ()` = 어떤 명제든 참은 도출 가능
- 커링: `((a, b) → c) ≅ (a → (b → c))` = "a ∧ b ⇒ c"와 "a ⇒ (b ⇒ c)"는 동치

이 대응은 단순한 비유가 아니라 **구조적 동형**이다. 직관주의 논리(intuitionistic logic)에서의 증명 체계와 타입이 있는 람다 계산법은 동일한 구조를 가진다.

---

## 정리

- **함수 객체**: 보편적 구성으로 정의. 평가 사상 `eval :: (a⇒b) × a → b`이 핵심
- **커링**: `(a × b → c) ≅ (a → (b → c))`. 보편적 구성의 factorizer
- **지수 표기**: 함수 타입 `a → b`를 `b^a`로 씀. 집합의 크기와 일치
- **데카르트 닫힌 카테고리(CCC)**: 끝 대상 + 곱 + 지수를 모두 갖춘 카테고리
- **타입 대수의 완성**: Void=0, Unit=1, Either=+, Pair=×, (→)=지수. 수의 법칙이 그대로 성립
- **Curry-Howard 동형**: 타입=명제, 프로그램=증명, 함수=함의. 논리와 타입 이론의 구조적 대응
