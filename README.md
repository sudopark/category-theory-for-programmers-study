# category-theory-for-programmers-study
프로그래머를 위한 카테고리 이론 스터디


출처: https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/

Bartosz Milewski의 [Category Theory for Programmers](https://github.com/hmemcpy/milewski-ctfp-pdf/releases/download/v1.3.0/category-theory-for-programmers.pdf)과 [유트부 영상](https://www.youtube.com/playlist?list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_)을 바탕으로 하는 스터디 자료 입니다.

-----

## Preface
: 저자는 프로그래머 + 엔지니어를 위한 카테고리 이론 책을 집필한다고 합니다. 컴퓨터 사이언스와 엔지니어링 두 분야 모두 일한 경험이 있는 저자입장에서는 두 집단의 격차가 매우 크다는것을 알게되었고 이를 설명하는 요구를 많이 받았다고 합니다. 

#### 당신에게 이 책이 필요한 이유는?
1. 카테고리 이론은 프로그래밍에 매우 유용합니다. 하스칼 프로그래머들은 이 장점을 오랜시간 동안 누려왔고 여러 언어에 이점이 반영되었습니다.
2. 세상엔 필요한 분야에따라 다양한 종류의 수학이 있고 당신은 대수학이나 계산에 거부감이 있을 수 있습니다. 하지만 카테고리 이론은 프로그래머에게 흥미를 끌 수 있는 이론입니다. 카테고리 이론은 세부사항(particulars)을 다루는 것이 아니라 구조(structure)를 다루는 이론이기 때문입니다. 이는 프로그램을 합성하기 쉬운 구조를 말합니다.
    - 합성은 카테고리 이론의 근본입니다. 이것은 카테고리 정의 자체의 일부이고, 프로그래밍의 본질은 합성이라고 강력하게 주장 할 수 있습니다.
        - 구조적 프로그래밍에서 합성가능한 코드블럭을 만들고 이를 합성 했었습니다.
        - 객체지향 언어는 객체 합성 그자체 입니다
        - 함수형 프로그래밍에서 함수와 수학적인 데이터 스트럭처를 합성하는것은 물론 동시성을 고려해야 하는 상황에서도 합성이 가능하게 합니다.
3. 쉽게 수학을 가르쳐준다고합니다🤩

#### 카테고리 이론을 배우려면 Haskell을 알아야하나? →  No
책을 읽기에 앞서 당신은 함수형 언어(haskell)을 먼저 알아야하나 생각할수도 있지만 이는 카테고리 이론이 함수형 프로그래밍에만 적용된다고 말하는 것과 같습니다. 이론은 보다 범용적이기 때문에 저자는 C++를 통해 예제를 들것이라 합니다.

#### 안배워도 코드짤 수 있는데? → No
당신은 이미 카테고리 이론이나 함수형 프로그래밍 없이 많은 경력을 쌓아왔고 그거 없어도 잘할 수 있는데? 라고 말할 수 있습니다. 하지만 명령형 언어나 객체지향 언어에서도 함수형 피쳐들이 추가되고 있습니다. 이것은 시대적인 흐름입니다. 서서히 끓는 물 안에서 모른체 헤엄치는 개구리가 되지는 마세요.

이 추세의 가장 큰 이유는 멀티코어의 발전에따라 부각되는 동시성입니다. 기본 전제가 데이터 캡슐화인 객체지향언어에 공유와 mutation이 추가된다면 레이스 컨디션의 주된 타겟이 됩니다. mutex를 이용하여 보호되는 데이터는 나름 유용하다 하지만 lock은 결합을 방해하고 데드락을 빈번하게 발생시키며 디버깅도 어렵게한다.

동시성을 제외한다고 하여도 증가하는 소프트웨어의 복잡성은 명령형 프로그래밍의 확장성 한계를 시험하고 있습니다.  쉽게말해 사이드 이펙트가 통제 불가능해지고 있습니다. 사이드 이펙트 자체가 나쁜것은 아닙니다 하지만 사이드 이팩트는 큰 스케일의 코드를 관리하는것을 매우 어렵게합니다.

----

# 목차
## Part One
- [1. Category: The Essence of Composition](Part1/1_Category-The_Essence_of_Composition.md)
- [2. Types and Functions](Part1/2_Type_and_Fcuntions.md)
- [3. Categories Great and Small](Part1/3_Categories_Great_and_Small.md)
- [4. Kleisli Categories](Part1/4_Kleisli_Categories.md)
- [5. Products and Coproducts](Part1/5_Products_and_Coproducts.md)
- [6. Simple Algebric Data Types](Part1/6_Simple_Algebric_Data_Types.md)
- [7. Functors](Part1/7_Functors.md)
- [8. Functoriality](Part1/8_Functoriality.md)