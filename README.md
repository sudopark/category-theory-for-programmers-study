# Category Theory for Programmers — 스터디 노트

Bartosz Milewski의 [Category Theory for Programmers](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)를 기반으로 한 한국어 스터디 노트입니다.

원서의 전체 31장을 한국어로 요약하고, Haskell/Swift 코드 예제를 포함합니다.

---

## Part One — 기초 카테고리 이론

| # | 제목 | 핵심 |
|---|------|------|
| 1 | [Category: The Essence of Composition](Part1/1_Category-The_Essence_of_Composition.md) | 합성, 항등, 결합법칙 |
| 2 | [Types and Functions](Part1/2_Type_and_Fcuntions.md) | 타입=집합, 순수함수, Set 카테고리 |
| 3 | [Categories Great and Small](Part1/3_Categories_Great_and_Small.md) | 순서, 모노이드, 홈셋 |
| 4 | [Kleisli Categories](Part1/4_Kleisli_Categories.md) | Writer, 장식된 함수, fish 연산자 |
| 5 | [Products and Coproducts](Part1/5_Products_and_Coproducts.md) | 보편적 구성, 쌍대성 |
| 6 | [Simple Algebraic Data Types](Part1/6_Simple_Algebric_Data_Types.md) | 곱 타입, 합 타입, 타입의 대수 |
| 7 | [Functors](Part1/7_Functors.md) | fmap, 펑터 법칙, 펑터 합성 |
| 8 | [Functoriality](Part1/8_Functoriality.md) | 쌍펑터, 공변/반변, 프로펑터 |
| 9 | [Function Types](Part1/9_Function_Types.md) | 지수, 커링, CCC, Curry-Howard |
| 10 | [Natural Transformations](Part1/10_Natural_Transformations.md) | 펑터 간 사상, 자연성, 2-카테고리 |

## Part Two — 범주적 구성

| # | 제목 | 핵심 |
|---|------|------|
| 11 | [Declarative Programming](Part2/11_Declarative_Programming.md) | 선언적 vs 명령적, 보편적 구성 |
| 12 | [Limits and Colimits](Part2/12_Limits_and_Colimits.md) | 뿔, 극한, 쌍대극한 |
| 13 | [Free Monoids](Part2/13_Free_Monoids.md) | 자유 구성, 망각 펑터, 수반 |
| 14 | [Representable Functors](Part2/14_Representable_Functors.md) | 홈 펑터, tabulate/index |
| 15 | [The Yoneda Lemma](Part2/15_The_Yoneda_Lemma.md) | 요네다 보조정리, CPS |
| 16 | [Yoneda Embedding](Part2/16_Yoneda_Embedding.md) | 충실가득 펑터, presheaf |

## Part Three — 고급 카테고리 이론

| # | 제목 | 핵심 |
|---|------|------|
| 17 | [It's All About Morphisms](Part3/17_Its_All_About_Morphisms.md) | 사상 중심 복습 |
| 18 | [Adjunctions](Part3/18_Adjunctions.md) | 수반, 단위/쌍대단위, 삼각 항등식 |
| 19 | [Free/Forgetful Adjunctions](Part3/19_Free_Forgetful_Adjunctions.md) | 자유/망각 펑터, 자유 모나드 |
| 20 | [Monads: Programmer's Definition](Part3/20_Monads_Programmers_Definition.md) | bind, return, 모나드 법칙 |
| 21 | [Monads and Effects](Part3/21_Monads_and_Effects.md) | Maybe, State, Writer, Reader |
| 22 | [Monads Categorically](Part3/22_Monads_Categorically.md) | η, μ, 자기 함자의 모노이드 |
| 23 | [Comonads](Part3/23_Comonads.md) | extract, duplicate, Store |
| 24 | [F-Algebras](Part3/24_F_Algebras.md) | 시작 대수, Fix, 카타모피즘 |
| 25 | [Algebras for Monads](Part3/25_Algebras_for_Monads.md) | T-대수, Eilenberg-Moore |
| 26 | [Ends and Coends](Part3/26_Ends_and_Coends.md) | forall/exists, 닌자 요네다 |
| 27 | [Kan Extensions](Part3/27_Kan_Extensions.md) | 좌/우 칸 확장, 모든 것의 일반화 |
| 28 | [Enriched Categories](Part3/28_Enriched_Categories.md) | V-카테고리, 거리 공간 |
| 29 | [Topoi](Part3/29_Topoi.md) | 부분 대상 분류자, 직관주의 논리 |
| 30 | [Lawvere Theories](Part3/30_Lawvere_Theories.md) | 대수적 이론, 로비어 vs 모나드 |
| 31 | [Monads, Monoids, and Categories](Part3/31_Monads_Monoids_and_Categories.md) | 이중 카테고리, 합성의 본질 |

---

## 참고 자료

- [원서 PDF](https://github.com/hmemcpy/milewski-ctfp-pdf/releases/download/v1.3.0/category-theory-for-programmers.pdf)
- [YouTube 강의](https://www.youtube.com/playlist?list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_)
- [블로그 원문](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)
