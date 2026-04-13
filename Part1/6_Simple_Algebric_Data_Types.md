# Chapter 6: Simple Algebraic Data Types — 단순 대수적 데이터 타입

> **핵심 개념**: 곱 타입(product type)과 합 타입(sum type)을 결합하면 거의 모든 데이터 구조를 표현할 수 있다. 타입의 대수에서 곱은 곱셈, 합은 덧셈, Void는 0, Unit은 1에 대응하며, 이 비유는 재귀적 데이터 구조의 정의까지 확장된다.

---

## 카테고리 정의에서 타입 구현으로

Ch5에서 곱과 쌍대곱을 **카테고리 일반의 추상적 정의**로 배웠다 — 투영/주입과 보편적 성질만으로, 내부 구조 없이 정의했다. 이 장에서는 그 정의가 **Set 카테고리(= 프로그래밍의 타입 세계)**에서 어떤 구체적 구현이 되는지를 다룬다.

| 카테고리 일반 (Ch5) | Set/프로그래밍에서의 구현 (이 장) |
|---------------------|--------------------------------|
| 곱 (product) | 페어, 튜플, 레코드/구조체 |
| 쌍대곱 (coproduct) | Either, enum, Optional |
| 시작 대상 (initial) | Void |
| 끝 대상 (terminal) | Unit `()` |

이 장의 핵심 발견은, 이 구현들을 조합하면 **숫자의 덧셈/곱셈과 동일한 대수 법칙**이 성립한다는 것이다.

---

## 곱 타입 (Product Types)

프로그래밍에서 두 타입의 곱은 **페어(pair)**로 나타낸다.

```haskell
swap :: (a, b) -> (b, a)
swap (x, y) = (y, x)
```

`(Int, Bool)`과 `(Bool, Int)`는 프로그래밍에서는 다른 타입이지만, `swap`을 통해 **동형(isomorphic)**이다. 이 동형은 **Set 카테고리 레벨**의 이야기다. Set 카테고리에서 동형사상은 곧 **전단사 함수(bijection)** — 모든 원소가 1:1로 대응된다는 뜻이다. `swap . swap = id`이므로 `swap`은 동형사상이고, 두 타입은 집합으로서 같은 크기와 구조를 가진다.

그런데 왜 둘 다 `Int`와 `Bool`의 곱이 될 수 있을까? Ch5에서 곱은 **투영과 보편적 성질**로 정의했지, "순서쌍"이라는 구체적 구현으로 정의한 게 아니기 때문이다. `(Int, Bool)`도, `(Bool, Int)`도 보편적 성질을 만족하므로 둘 다 곱이다. 그리고 Ch5의 시작 대상 유일성 증명과 같은 논리로, 보편적 성질을 만족하는 대상이 여럿 있으면 **반드시 동형**이다. 프로그래밍에서는 다른 타입이지만, 카테고리 이론에서는 **본질적으로 같은 곱**이다.

중첩된 페어(nested pair)도 마찬가지로 동형이다:

```haskell
-- 결합법칙과 유사
alpha :: ((a, b), c) -> (a, (b, c))
alpha ((x, y), z) = (x, (y, z))

alpha_inv :: (a, (b, c)) -> ((a, b), c)
alpha_inv (x, (y, z)) = ((x, y), z)
```

Unit `()`과의 곱은 정보를 추가하지 않는다 — 곱셈에서 1을 곱하는 것과 같다:

```haskell
-- (a, ()) ≅ a
rho :: (a, ()) -> a
rho (x, ()) = x

rho_inv :: a -> (a, ())
rho_inv x = (x, ())
```

정리하면, 페어로 타입을 합칠 때 세 가지 성질이 성립한다:
- **교환**: `(a, b) ≅ (b, a)` — 순서를 바꿔도 같은 정보
- **결합**: `((a, b), c) ≅ (a, (b, c))` — 묶는 방식을 바꿔도 같은 정보
- **단위**: `(a, ()) ≅ a` — Unit을 곱해도 정보가 늘지 않음

이것은 Ch3에서 배운 **모노이드**의 구조와 같다. 이항 연산이 "곱 타입 만들기"이고 단위 원소가 `()`인 모노이드다. (단, 등호가 아니라 **동형**으로 성립하므로, 엄밀히는 **모노이드 카테고리(monoidal category)**라 부른다.)

### 레코드 (Records)

튜플의 각 컴포넌트에 이름을 붙이면 **레코드**(Haskell) 또는 **구조체**(C/Swift)가 된다:

```haskell
data Element = Element { name         :: String
                       , symbol       :: String
                       , atomicNumber :: Int }

-- 레코드와 튜플은 동형
tupleToElem :: (String, String, Int) -> Element
tupleToElem (n, s, a) = Element { name = n, symbol = s, atomicNumber = a }

-- 필드 이름은 접근 함수로도 쓰인다
startsWithSymbol :: Element -> Bool
startsWithSymbol e = symbol e `isPrefixOf` name e
```

```swift
struct Element {
    let name: String
    let symbol: String
    let atomicNumber: Int
}
```

---

## 합 타입 (Sum Types)

카테고리의 쌍대곱(coproduct)은 프로그래밍에서 **합 타입(sum type)**이 된다.

```haskell
data Either a b = Left a | Right b
```

곱 타입과 마찬가지로 Either도 교환 가능하고(동형) 중첩 순서도 무관(동형)하다.

곱 타입에서 `()`가 1 역할(곱해도 변하지 않음)을 했듯이, 합 타입에서 `Void`는 **0의 역할**을 한다:

```haskell
-- Either a Void ≅ a
-- Void 타입의 값은 존재하지 않으므로 Right를 만들 수 없다.
-- 따라서 Either a Void의 값은 반드시 Left x 형태이고,
-- 이는 a 값을 감싸고 있을 뿐이다.

-- 덧셈으로 보면: a + 0 = a
```

### Maybe (Optional)

"값이 있거나 없다"를 표현하는 가장 간단한 합 타입:

```haskell
data Maybe a = Nothing | Just a
-- 이것은 사실 Either () a와 동형이다
```

```swift
enum Optional<T> {
    case none
    case some(T)
}
```

### 리스트 (List) — 재귀적 합 타입

```haskell
data List a = Nil | Cons a (List a)
-- 비어있거나(Nil), 머리(head)와 꼬리(tail)로 구성
```

Haskell의 데이터 구조는 **불변(immutable)**이다. 생성자로 만든 값은 변하지 않으며, 패턴 매칭으로 해체(deconstruction)할 수 있다:

```haskell
maybeTail :: List a -> Maybe (List a)
maybeTail Nil        = Nothing
maybeTail (Cons _ t) = Just t
```

---

## 타입의 대수학 (Algebra of Types)

곱 타입과 합 타입을 결합하면 **타입의 대수**가 나타난다. 놀랍게도 이 대수는 숫자의 덧셈/곱셈과 정확히 대응된다:

| 숫자 | 타입 |
|------|------|
| 0 | `Void` |
| 1 | `()` (Unit) |
| a + b | `Either a b` |
| a × b | `(a, b)` |
| 2 = 1 + 1 | `Bool = True \| False` |
| 1 + a | `Maybe a = Nothing \| Just a` |

이 대응 관계는 실제 대수학 법칙을 따른다:

### a × 0 = 0
```haskell
-- (a, Void) ≅ Void — Void 값을 만들 수 없으므로 페어도 만들 수 없다
```

### 분배법칙: a × (b + c) = a × b + a × c
```haskell
prodToSum :: (a, Either b c) -> Either (a, b) (a, c)
prodToSum (x, e) = case e of
    Left  y -> Left  (x, y)
    Right z -> Right (x, z)

sumToProd :: Either (a, b) (a, c) -> (a, Either b c)
sumToProd e = case e of
    Left  (x, y) -> (x, Left y)
    Right (x, z) -> (x, Right z)
```

### 재귀적 타입 방정식

리스트의 정의를 대수적으로 쓰면:

```
List a = Nil | Cons a (List a)
       = ()  | (a, List a)
       = 1 + a × List a
```

`Nil`은 값이 하나뿐이니 `1`(Unit), `Cons`는 원소 하나(`a`)와 나머지 리스트(`List a`)의 곱이다.

x = List a로 놓으면: `x = 1 + a·x`

이 방정식의 x에 오른쪽을 계속 대입해보자:

```
x = 1 + a·x
  = 1 + a·(1 + a·x)           -- x를 대입
  = 1 + a + a²·x              -- 분배법칙으로 전개
  = 1 + a + a²·(1 + a·x)      -- 다시 대입
  = 1 + a + a² + a³·x         -- 전개
  = ...
  = 1 + a + a² + a³ + a⁴ + ...
```

각 항의 의미:
- `1` — 빈 리스트 `[]`
- `a` — 원소 1개 `[x]`
- `a²` = `(a, a)` — 원소 2개 `[x, y]`
- `a³` = `(a, a, a)` — 원소 3개 `[x, y, z]`
- ...

"리스트는 비어있거나, 원소가 1개이거나, 2개이거나, ..."라는 직관과 정확히 일치한다.

> 이처럼 타입을 기호 변수로 놓고 대수적 방정식처럼 다룰 수 있기 때문에 **대수적 데이터 타입(Algebraic Data Type, ADT)**이라 부른다.

### 논리와의 대응 (Curry-Howard)

타입의 대수는 산술뿐 아니라 **논리**와도 대응된다:

| 논리 | 타입 | 직관 |
|------|------|------|
| False | `Void` | 거짓 — 증명(값)이 존재하지 않는 명제 |
| True | `()` | 참 — 증명(값)이 항상 존재하는 명제 |
| a ∧ b (AND) | `(a, b)` | a와 b 둘 다 있어야 한다 |
| a ∨ b (OR) | `Either a b` | a 또는 b 중 하나가 있다 |

핵심 아이디어는 **"타입은 명제이고, 그 타입의 값은 증명이다"**라는 것이다:

- `Void`에는 값이 없다 → 거짓 명제에는 증명이 없다
- `()`에는 값이 하나 있다 → 참 명제에는 증명이 존재한다
- `(a, b)` 값을 만들려면 a 값과 b 값이 **둘 다** 필요하다 → "a AND b"를 증명하려면 a의 증명과 b의 증명이 모두 필요하다
- `Either a b` 값을 만들려면 a 값이나 b 값 **하나만** 있으면 된다 → "a OR b"를 증명하려면 둘 중 하나의 증명이면 충분하다

이것이 **Curry-Howard 동형(isomorphism)**의 기초다. 여기서는 곱=AND, 합=OR까지만 다루지만, Ch9에서 함수 타입을 배우면 `a → b` = "a이면 b이다"(함의, implication)까지 대응이 확장되어 완성된다.

---

## 정리

- **곱 타입**: 페어, 튜플, 레코드/구조체. 여러 타입의 값을 **모두** 보유 (AND)
- **합 타입**: Either, Maybe, 열거형. 여러 타입 중 **하나**의 값을 보유 (OR)
- **타입의 대수**: Void=0, Unit=1, Either=+, Pair=×. 분배법칙 등 대수 법칙 성립
- 재귀적 데이터 구조(List 등)는 타입 방정식의 해로 표현 가능
- 불변 데이터 + 패턴 매칭 = 생성자로 만들고 패턴으로 해체
- **Curry-Howard 동형**: 타입은 논리적 명제에, 곱=AND, 합=OR에 대응

---

## 참고: 타입의 대수는 반환(semiring)이다

타입의 대수가 숫자의 대수와 어디까지 대응되는지 정리하면:

| 숫자의 연산 | 타입의 연산 | 대응 여부 |
|------------|-----------|----------|
| 덧셈 | `Either a b` (합 타입) | ✓ |
| 곱셈 | `(a, b)` (곱 타입) | ✓ |
| 0 | `Void` | ✓ |
| 1 | `()` | ✓ |
| 분배법칙 | `(a, Either b c) ≅ Either (a,b) (a,c)` | ✓ |
| 뺄셈 | ??? | ✗ |

"Int에서 Bool을 빼면 무슨 타입?"이라는 질문은 성립하지 않는다. 타입에는 **뺄셈에 해당하는 연산이 없다**.

수학에서 덧셈·곱셈·뺄셈이 있고 분배법칙이 성립하는 구조를 **고리(ring)**라 한다. 타입의 대수는 여기서 뺄셈만 빠져 있으므로 "semi(반)" + "ring(환)" = **반환(semiring)**이라 부른다. ring에서 **n**egation(부정/뺄셈)을 뺐다는 뜻으로 **rig**(ring − n)이라고도 한다.
