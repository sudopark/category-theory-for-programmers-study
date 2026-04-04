# Chapter 4: Kleisli Categories — 클라이슬리 카테고리

> **핵심 개념**: 순수 함수로 사이드 이펙트를 모델링하려면 어떻게 할까? 함수의 반환값에 부가 정보를 덧붙이는 **장식된 함수(embellished function)**를 사상으로, 이들의 합성을 재정의하면 새로운 카테고리가 만들어진다. 이것이 **클라이슬리 카테고리(Kleisli category)**이며, **모나드(Monad)**의 첫 맛보기다.

---

## 문제: 순수 함수로 로깅하기

함수의 실행을 로깅하고 싶다고 하자. 명령형 스타일에서는 전역 변수를 쓸 수 있다:

```swift
var logger: String = ""

func negate(_ b: Bool) -> Bool {
    logger += "Not so! "
    return !b
}
```

이 함수는 외부 상태를 변경하므로 **순수 함수가 아니다**. 순수하게 만들려면 로그를 입출력에 포함시켜야 한다:

```swift
func negate(_ b: Bool, logger: String) -> (Bool, String) {
    return (!b, logger + "Not so! ")
}
```

하지만 이러면 로그 히스토리를 계속 들고 다녀야 하고, 인터페이스가 복잡해진다. 핵심은 negate의 주된 목적은 Bool을 뒤집는 것이고, 로깅은 **부가적인 것**이라는 점이다.

---

## Writer 타입으로 합성 추상화하기

반환값에만 로그를 붙이는 방식으로 개선해보자. `Writer`라는 튜플 타입을 도입한다:

```swift
typealias Writer<A> = (A, String)

func toUpper(_ s: String) -> Writer<String> {
    return (s.uppercased(), "toUpper ")
}

func toWords(_ s: String) -> Writer<[String]> {
    return (s.components(separatedBy: " "), "toWords ")
}

// 합성: toUpper 후 toWords
func process(_ s: String) -> Writer<[String]> {
    let p1 = toUpper(s)
    let p2 = toWords(p1.0)
    return (p2.0, p1.1 + p2.1)  // 결과값 + 로그 누적
}
```

합성의 패턴이 보인다:
1. 첫 번째 함수를 실행
2. 결과 튜플에서 값을 꺼내 두 번째 함수에 전달
3. 두 로그 문자열을 이어 붙임
4. 새 튜플 반환

이 패턴을 고차 함수로 추상화할 수 있다:

```swift
func compose<A, B, C>(
    _ m1: @escaping (A) -> Writer<B>,
    _ m2: @escaping (B) -> Writer<C>
) -> (A) -> Writer<C> {
    return { a in
        let p1 = m1(a)
        let p2 = m2(p1.0)
        return (p2.0, p1.1 + p2.1)
    }
}
```

항등 사상도 필요하다 — 값을 그대로 반환하고 빈 로그를 붙이는 함수:

```swift
func identity<A>(_ a: A) -> Writer<A> {
    return (a, "")
}
```

이렇게 정의한 합성과 항등 사상은 카테고리의 요구사항을 충족한다:
- **결합법칙**: 값의 합성은 일반 함수 합성이므로 결합적이고, 로그의 문자열 연결도 결합적이다
- **항등 사상**: 빈 문자열은 연결의 단위 원소

> 기민한 독자라면 이 구조가 **String 모노이드**를 사용하고 있음을 눈치챘을 것이다. compose에서는 `mappend`(문자열 연결)를, identity에서는 `mempty`(빈 문자열)를 수행한다. 이것은 **어떤 모노이드로든 일반화**할 수 있다.

---

## Haskell에서의 Writer

```haskell
type Writer a = (a, String)

-- "fish" 연산자로 합성 정의
(>=>) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)
m1 >=> m2 = \x ->
    let (y, s1) = m1 x
        (z, s2) = m2 y
    in (z, s1 ++ s2)

-- 항등 사상
return :: a -> Writer a
return x = (x, "")
```

이제 합성이 깔끔해진다:

```haskell
upCase :: String -> Writer String
upCase s = (map toUpper s, "upCase ")

toWords :: String -> Writer [String]
toWords s = (words s, "toWords ")

-- fish 연산자로 합성
process :: String -> Writer [String]
process = upCase >=> toWords
```

---

## 클라이슬리 카테고리

우리가 만든 것은 새로운 종류의 카테고리가 아니라, **모나드를 기반으로 하는 카테고리**인 **클라이슬리 카테고리(Kleisli category)**의 한 예다.

클라이슬리 카테고리의 구조:

- **대상**: 프로그래밍 언어의 타입들 (기존과 동일)
- **사상**: A에서 B로의 사상은 `A → m B` 형태의 함수 (m은 장식/embellishment)
- **합성**: fish 연산자 `>=>`로 정의. 단순히 반환값을 다음 함수에 넣는 것 이상의 로직 포함
- **항등 사상**: `return` — 값을 장식으로 감싸기만 함

> **핵심 통찰**: 사상 `A → Writer B`에서 Writer라는 장식이 붙었지만, 카테고리적으로 이 사상은 여전히 A에서 B로의 화살표다. 장식은 합성 방식을 변경할 뿐, 대상 간의 관계는 유지된다.

이 장에서 사용한 Writer는 하나의 예시일 뿐이다. **장식(embellishment)**이라는 용어는 나중에 카테고리 이론의 **자기 함자(endofunctor)**에 대응된다는 것을 알게 될 것이다. 그리고 클라이슬리 카테고리를 정의하기 위한 장식의 조건이 바로 **모나드**다.

순수 함수만으로 사이드 이펙트를 모델링하는 이 접근법은 매우 강력하다. 이전에는 타입과 함수를 Set 카테고리로 모델링했다면, 이번에는 모델을 **약간 다른 카테고리로 확장**한 것이다. 합성 자체를 가지고 놀 수 있는 자유도가 한 차원 늘었다.

---

## 정리

- **장식된 함수(embellished function)**: `A → Writer B`처럼 반환값에 부가 정보를 덧붙인 함수
- **클라이슬리 카테고리**: 대상은 타입, 사상은 장식된 함수, 합성은 fish 연산자 `>=>`
- **fish 연산자** `>=>`: 장식된 함수들의 합성을 정의 — 값 전달 + 부가 정보 결합
- **return**: 클라이슬리 카테고리의 항등 사상 — 값을 장식으로 감쌈
- Writer의 로그 결합은 **모노이드의 mappend**, return의 빈 로그는 **mempty**에 대응
- 이 구조는 **모나드(Monad)**의 첫 번째 예시이며, 순수 함수로 사이드 이펙트를 표현하는 일반적 메커니즘
