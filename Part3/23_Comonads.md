# Chapter 23: Comonads — 코모나드

> **핵심 개념**: **코모나드(Comonad)**는 모나드의 **쌍대(dual)**로, 화살표를 뒤집어 얻는다. 모나드가 값을 **감싸고(return)** 풀어내는(join) 구조라면, 코모나드는 값을 **추출하고(extract)** 복제하는(duplicate) 구조다. 프로그래밍에서 코모나드는 **문맥이 있는 계산**, **셀룰러 오토마타**, **스트림 처리** 등에 활용된다.

---

## 코모나드의 정의

모나드의 각 요소를 뒤집으면:

| 모나드 | 코모나드 |
|--------|---------|
| η: Id → T (return) | ε: W → Id (**extract**) |
| μ: T² → T (join) | δ: W → W² (**duplicate**) |
| `>>=` :: m a → (a → m b) → m b | `=>>` :: w a → (w a → b) → w b (**extend**) |

```haskell
class Functor w => Comonad w where
    extract   :: w a -> a            -- 문맥에서 값을 추출
    duplicate :: w a -> w (w a)      -- 문맥을 복제
    extend    :: (w a -> b) -> w a -> w b  -- 문맥 있는 함수를 적용
    
    -- extend는 duplicate + fmap으로 정의 가능:
    extend f = fmap f . duplicate
```

---

## 코모나드 법칙

모나드 법칙을 뒤집으면:

```haskell
-- extract는 duplicate의 왼쪽/오른쪽 역
extract . duplicate      = id
fmap extract . duplicate = id

-- duplicate의 결합법칙
duplicate . duplicate = fmap duplicate . duplicate
```

---

## Co-Kleisli 카테고리

모나드가 클라이슬리 화살표 `a → m b`를 합성하듯, 코모나드는 **Co-Kleisli 화살표** `w a → b`를 합성한다:

```haskell
-- Co-Kleisli 합성
(=>=) :: (w a -> b) -> (w b -> c) -> (w a -> c)
f =>= g = g . extend f
-- 또는: g . fmap f . duplicate
```

직관: Co-Kleisli 화살표는 "문맥 전체를 보고 값 하나를 내놓는 함수"다.

---

## 대표적 코모나드

### Product 코모나드 (Env / Reader의 쌍대)

```haskell
data Env e a = Env e a  -- (환경, 값) 쌍

instance Comonad (Env e) where
    extract (Env _ a) = a                     -- 값 추출
    duplicate (Env e a) = Env e (Env e a)     -- 환경 복제
```

Reader 모나드가 "환경에 접근하는 계산"이라면, Env 코모나드는 "환경이 붙어 있는 값"이다.

### Stream 코모나드

```haskell
data Stream a = Cons a (Stream a)

instance Comonad Stream where
    extract (Cons a _) = a                           -- 현재 값
    duplicate s@(Cons _ rest) = Cons s (duplicate rest)  -- 모든 접미사의 스트림
```

`duplicate`는 스트림의 모든 "미래 시점"을 만들어낸다. `extend f`는 각 시점에서 `f`를 적용한다 — **스트림 처리의 핵심**이다.

### Store 코모나드 (State의 쌍대)

```haskell
data Store s a = Store (s -> a) s  -- (위치 → 값 함수, 현재 위치)

instance Comonad (Store s) where
    extract (Store f s) = f s                        -- 현재 위치의 값
    duplicate (Store f s) = Store (Store f) s         -- 모든 위치를 "현재 위치"로 보기
```

Store 코모나드는 **셀룰러 오토마타**와 **이미지 처리의 컨볼루션** 등에 활용된다.

---

## 코모나드의 직관

- **모나드**: 값을 문맥 안에 **넣고**, 문맥 안에서 작업 (`return`, `>>=`)
- **코모나드**: 문맥에서 값을 **꺼내고**, 문맥을 활용하여 작업 (`extract`, `extend`)

모나드가 "어떻게 효과를 합성할 것인가"의 문제라면, 코모나드는 "주어진 문맥(이웃, 히스토리, 환경)을 어떻게 활용할 것인가"의 문제다.

---

## 정리

- **코모나드** = 모나드의 쌍대. 화살표를 뒤집어 얻음
- **extract**: 문맥에서 값을 추출 (return의 쌍대)
- **duplicate**: 문맥을 복제 `w a → w (w a)` (join의 쌍대)
- **extend**: Co-Kleisli 화살표 `w a → b`를 적용
- **코모나드 법칙**: extract/duplicate 관계 + duplicate 결합법칙
- **Product (Env)**: 환경이 붙은 값. Reader의 쌍대
- **Stream**: 무한 스트림. extend로 스트림 처리
- **Store**: 위치 기반 접근. 셀룰러 오토마타, 이미지 처리
