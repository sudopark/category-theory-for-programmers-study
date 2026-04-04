# Chapter 6: Simple Algebraic Data Types — 단순 대수적 데이터 타입

> **핵심 개념**: 곱 타입(product type)과 합 타입(sum type)을 결합하면 거의 모든 데이터 구조를 표현할 수 있다. 타입의 대수에서 곱은 곱셈, 합은 덧셈, Void는 0, Unit은 1에 대응하며, 이 비유는 재귀적 데이터 구조의 정의까지 확장된다.

---

## 곱 타입 (Product Types)

프로그래밍에서 두 타입의 곱은 **페어(pair)**로 나타낸다.

```haskell
swap :: (a, b) -> (b, a)
swap (x, y) = (y, x)
```

`(Int, Bool)`과 `(Bool, Int)`는 타입으로는 다르지만, `swap`을 통해 **동형(isomorphic)**이다 — 같은 정보를 다른 형식으로 담고 있을 뿐이다.

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

> Set은 곱에 대해 **모노이드 카테고리(monoidal category)**를 이룬다: 이항 연산 = 곱, 단위 원소 = Unit

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

합 타입에서 `Void`는 0의 역할: `Either a Void ≅ a` — Right를 만들 수 없으므로 Left만 가능하고, 이는 a를 감싸는 것에 불과하다.

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
List a = 1 + a × List a
```

x = List a로 놓으면: `x = 1 + a·x`

오른쪽의 x를 계속 대입하면:

```
x = 1 + a + a² + a³ + a⁴ + ...
```

이는 "리스트는 비어있거나(1), 원소 1개이거나(a), 2개이거나(a²), ..." 라는 직관과 정확히 일치한다.

> 기호 변수로 방정식을 풀 수 있다는 점에서 이런 타입을 **대수적 데이터 타입(Algebraic Data Type, ADT)**이라 부른다.

### 논리와의 대응 (Curry-Howard)

타입의 대수는 논리와도 대응된다:

| 논리 | 타입 |
|------|------|
| False | `Void` |
| True | `()` |
| a ∨ b (OR) | `Either a b` |
| a ∧ b (AND) | `(a, b)` |

곱 타입은 a **그리고** b를 모두 가져야 하고(AND), 합 타입은 a **또는** b 중 하나를 가진다(OR). 이것이 **Curry-Howard 동형(isomorphism)**의 기초이며, 함수 타입을 배운 후에 더 깊이 다룬다.

---

## 정리

- **곱 타입**: 페어, 튜플, 레코드/구조체. 여러 타입의 값을 **모두** 보유 (AND)
- **합 타입**: Either, Maybe, 열거형. 여러 타입 중 **하나**의 값을 보유 (OR)
- **타입의 대수**: Void=0, Unit=1, Either=+, Pair=×. 분배법칙 등 대수 법칙 성립
- 재귀적 데이터 구조(List 등)는 타입 방정식의 해로 표현 가능
- 불변 데이터 + 패턴 매칭 = 생성자로 만들고 패턴으로 해체
- **Curry-Howard 동형**: 타입은 논리적 명제에, 곱=AND, 합=OR에 대응
- 이 구조들은 **반환(semiring/rig)**을 이루며, 타입 빼기(subtraction)만 없다
