# Chapter 20: Monads: Programmer's Definition — 모나드: 프로그래머의 정의

> **핵심 개념**: **모나드(Monad)**는 장식된 함수(embellished function)의 합성법이다. 클라이슬리 화살표 `a → m b`를 합성할 수 있게 해주는 구조로, `return`(값을 감싸기)과 `>>=`(bind, 값을 꺼내서 다음 함수에 전달)로 정의된다. 모나드 법칙은 이 합성이 카테고리의 공리를 만족하도록 보장한다.

---

## 모나드의 본질: 합성

모나드에 대한 온갖 신비주의적 비유(부리또, 우주복...)는 오해의 산물이다. 모나드의 본질은 단순하다: **합성(composition)**이다.

Ch.4에서 클라이슬리 카테고리를 배웠다. 장식된 함수 `a → m b`를 합성하려면:

1. 첫 함수의 결과에서 값을 꺼내기
2. 그 값을 두 번째 함수에 전달
3. 장식(context)을 합치기

이 과정을 추상화한 것이 **모나드**다.

---

## 세 가지 동등한 정의

### 1. Fish 연산자 (Kleisli 합성)

```haskell
(>=>) :: (a -> m b) -> (b -> m c) -> (a -> m c)
return :: a -> m a
```

### 2. Bind 연산자

```haskell
(>>=) :: m a -> (a -> m b) -> m b
return :: a -> m a

-- fish는 bind로 정의 가능:
f >=> g = \a -> f a >>= g
```

### 3. join + fmap

```haskell
join   :: m (m a) -> m a
return :: a -> m a
fmap   :: (a -> b) -> m a -> m b

-- bind는 join과 fmap으로 정의 가능:
ma >>= f = join (fmap f ma)
```

세 정의는 모두 동등하며, 상황에 따라 편한 것을 사용한다.

---

## 모나드 법칙

모나드가 클라이슬리 카테고리의 합성 규칙이 되려면 **카테고리의 공리**를 만족해야 한다:

### Fish 연산자 기준

```haskell
-- 왼쪽 항등: return >=> f = f
-- 오른쪽 항등: f >=> return = f  
-- 결합법칙: (f >=> g) >=> h = f >=> (g >=> h)
```

### Bind 기준

```haskell
-- 왼쪽 항등
return a >>= f  =  f a

-- 오른쪽 항등
ma >>= return  =  ma

-- 결합법칙
(ma >>= f) >>= g  =  ma >>= (\a -> f a >>= g)
```

### join 기준

```haskell
-- join과 return의 관계
join . return      = id        -- return으로 감싼 것을 join하면 원래대로
join . fmap return = id        -- 안쪽을 return으로 감싼 것을 join해도 원래대로
join . join = join . fmap join  -- 삼중 감싸기를 풀 때 순서 무관
```

---

## 대표적 모나드들

### Maybe 모나드

```haskell
instance Monad Maybe where
    return = Just
    Nothing >>= f = Nothing    -- 실패 전파
    Just x  >>= f = f x        -- 값 꺼내서 전달
```

### List 모나드

```haskell
instance Monad [] where
    return x = [x]
    xs >>= f = concatMap f xs  -- 각 원소에 f 적용 후 결과를 합침
```

### Writer 모나드

```haskell
instance Monoid w => Monad (Writer w) where
    return x = Writer (x, mempty)
    Writer (x, w) >>= f = let Writer (y, w') = f x
                           in Writer (y, w `mappend` w')
```

---

## do 표기법

Haskell의 `do` 표기법은 bind의 문법적 설탕이다:

```haskell
-- do 표기법
process :: String -> Writer [String]
process s = do
    upper <- toUpper s
    words <- toWords upper
    return words

-- 탈설탕(desugar)
process s = toUpper s >>= \upper ->
            toWords upper >>= \words ->
            return words
```

`do` 블록은 마치 명령형 코드처럼 보이지만, 실제로는 모나드의 bind를 연쇄적으로 호출하는 것이다.

---

## 정리

- **모나드** = 장식된 함수 `a → m b`의 합성법
- **세 가지 정의**: fish `>=>` / bind `>>=` / join + fmap — 모두 동등
- **return**: 값을 모나드 문맥으로 감싸기 (클라이슬리 항등 사상)
- **bind `>>=`**: 모나드에서 값을 꺼내 다음 장식된 함수에 전달
- **join**: 이중 감싸기를 풀기 `m (m a) → m a`
- **모나드 법칙**: 항등법칙(좌/우) + 결합법칙 — 합성이 카테고리를 형성하는 조건
- **do 표기법**: bind 연쇄의 문법적 설탕. 명령형처럼 보이지만 순수한 합성
