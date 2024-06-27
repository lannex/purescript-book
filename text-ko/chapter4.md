### 패턴 매칭

> 임시 주의: 이 장을 작업 중이라면, 2023년 11월에 4장과 5장이 바뀌었다는 점을 유의하십시오.

## 장의 목표

이 장에서는 새로운 개념 두 가지: 대수 데이터 타입과 패턴 매칭을 소개합니다. 또한 PureScript 타입 시스템의 흥미로운 기능인 행 다형성(row polymorphism)을 간단히 다룹니다.

패턴 매칭은 함수형 프로그래밍에서 흔히 사용되는 기법으로, 여러 가지 경우로 나누어 복잡한 아이디어를 간결하게 표현하는 함수를 작성할 수 있게 합니다.

대수 데이터 타입은 PureScript 타입 시스템의 특징으로, 타입의 언어에서 비슷한 수준의 표현력을 가능하게 합니다. 대수 데이터 타입은 패턴 매칭과 밀접하게 관련되어 있습니다.

이 장의 목표는 대수 타입과 패턴 매칭을 사용하여 단순한 벡터 그래픽을 설명하고 조작하는 라이브러리를 작성하는 것입니다.

## 프로젝트 설정

이 장의 소스 코드는 `src/Data/Picture.purs` 파일에 정의되어 있습니다.

`Data.Picture` 모듈은 간단한 도형을 위한 데이터 타입 `Shape`와 도형의 모음인 `Picture` 타입을 정의하며, 이 타입들을 다루기 위한 함수들도 함께 정의합니다.

이 모듈은 데이터 구조를 접는 함수를 제공하는 `Data.Foldable` 모듈을 가져옵니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:module_picture}}
```

`Data.Picture` 모듈은 또한 `Number` 모듈을 `as` 키워드를 사용하여 가져옵니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:picture_import_as}}
```

이로 인해 모듈 내의 타입과 함수를 사용할 수 있지만, 반드시 _qualified name_ 으로 사용해야 합니다. 예를 들어, `Number.max`와 같이 사용합니다. 이는 겹치는 가져오기를 피하거나 특정 모듈에서 가져온 것임을 명확히 하는 데 유용합니다.

> _참고_: 원래 모듈과 동일한 이름을 사용하여 가져오는 것은 필요하지 않으며, `import Data.Number as N`과 같이 더 짧은 이름을 사용하는 것이 일반적입니다.

## 간단한 패턴 매칭

예제를 살펴보겠습니다. 다음은 패턴 매칭을 사용하여 두 정수의 최대 공약수를 계산하는 함수입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:gcd}}
```

이 알고리즘은 유클리드 알고리즘이라고 불립니다. 온라인에서 정의를 검색하면 위 코드와 같은 수학적 방정식을 찾을 수 있을 것입니다. 패턴 매칭의 한 가지 장점은 단순하고 선언적인 코드를 작성하여 수학 함수 명세와 같은 코드를 정의할 수 있다는 것입니다.

패턴 매칭을 사용하여 작성된 함수는 조건 집합과 결과를 쌍으로 짝지어 동작합니다. 각 줄은 _대안_ 또는 _케이스_ 라고 합니다. 등호 왼쪽의 표현식은 _패턴_ 이라고 하며, 각 케이스는 공백으로 구분된 하나 이상의 패턴으로 구성됩니다. 케이스는 인수가 만족해야 하는 조건을 설명하며, 조건이 만족되면 등호 오른쪽의 표현식을 평가하여 반환합니다. 각 케이스는 순서대로 시도되며, 첫 번째로 조건을 만족하는 케이스가 반환 값을 결정합니다.

예를 들어, `gcd` 함수는 다음과 같은 단계로 평가됩니다:

- 첫 번째 케이스가 시도됩니다: 두 번째 인수가 0이면 함수는 첫 번째 인수 `n`을 반환합니다.
- 그렇지 않으면 두 번째 케이스가 시도됩니다: 첫 번째 인수가 0이면 함수는 두 번째 인수 `m`을 반환합니다.
- 그렇지 않으면 마지막 줄의 표현식을 평가하여 반환합니다.

패턴은 값을 이름에 바인딩할 수 있습니다. 예제의 각 줄은 인수 값을 `n`과 `m`이라는 이름에 바인딩합니다. 다른 패턴을 배우면서, 다른 패턴이 입력 인수에서 이름을 선택하는 다양한 방법과 대응된다는 것을 알게 될 것입니다.

## 간단한 패턴

위의 예제 코드에서는 두 가지 패턴 유형을 보여줍니다:

- 정수 리터럴 패턴: 값이 정확히 일치하는 경우에만 `Int` 타입의 값을 매칭합니다.
- 변수 패턴: 인수를 이름에 바인딩합니다.

다른 간단한 패턴 유형도 있습니다:

- `Number`, `String`, `Char`, `Boolean` 리터럴
- 와일드카드 패턴: 밑줄(`_`)로 표시되며, 모든 인수와 일치하고 이름을 바인딩하지 않습니다.

다음은 이러한 간단한 패턴을 사용하는 두 가지 예제입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:fromString}}

{{#include ../exercises/chapter4/src/ChapterExamples.purs:toString}}
```

이 함수들을 PSCi에서 시도해보세요.

## 가드

유클리드 알고리즘 예제에서, 우리는 `m > n` 및 `m <= n`일 때 두 대안을 전환하기 위해 `if .. then .. else` 표현식을 사용했습니다. 이 경우에 또 다른 옵션은 _가드_ 를 사용하는 것입니다.

가드는 패턴이 부과하는 제약 조건 외에 만족해야 하는 불리언 값 표현식입니다. 다음은 가드를 사용하여 다시 작성된 유클리드 알고리즘입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:gcdV2}}
```

이 경우, 세 번째 줄은 첫 번째 인수가 두 번째 인수보다 엄격히 큰 추가 조건을 부과하기 위해 가드를 사용합니다. 마지막 줄의 가드는 `otherwise` 표현식을 사용합니다. 이는 키워드처럼 보일 수 있지만, 실제로는 `Prelude`에서 정규 바인딩된 것입니다:

```text
> :type otherwise
Boolean

> otherwise
true
```

이 예제는 가드가 등호 기호 왼쪽에 나타나며, 파이프 문자(`|`)로 패턴 목록과 구분된다는 것을 보여줍니다.

## 연습 문제

1. (쉬움) 패턴 매칭을 사용하여 `factorial` 함수를 작성하세요. _힌트_: 0과 0이 아닌 입력의 두 모서리 사례를 고려하세요. _참고_: 이는 이전 장의 예제와 반복되지만, 여기서 다시 작성해보세요.
2. (중간) 다항식 \\( (1 + x)^n \\)의 \\( x^k \\) 항의 계수를 찾는 `binomial` 함수를 작성하세요. 이는 \\( n \\) 요소의 집합에서 \\( k \\) 요소의 부분 집합을 선택하는 방법의 수와 동일합니다. \\( n! / k!(n-k)! \\) 공식을 사용하세요. 여기서 \\( ! \\)는 이전에 작성한 팩토리얼 함수입니다. _힌트_: 코너 케이스를 처리하기 위해 패턴 매칭을 사용하세요. 완료하는 데 오랜 시간이 걸리거나 호출 스택에 대한 오류가 발생하면 더 많은 코너 케이스를 추가해보세요.
3. (중간) [_파스칼의 법칙_](https://en.wikipedia.org/wiki/Pascal%27s_rule)을 사용하여 이전 연습 문제와 동일한 이항 계수를 계산하는 `pascal` 함수를 작성하세요.

## 배열 패턴

_배열 리터럴 패턴_ 은 고정 길이의 배열을 매칭하는 방법을 제공합니다. 예를 들어, 빈 배열을 식별하는 `isEmpty` 함수를 작성하려면 첫 번째 대안에서 빈 배열 패턴(`[]`)을 사용할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:isEmpty}}
```

다음은 다섯 개의 요소를 다르게 바인딩하는 길이 다섯의 배열을 매칭하는 또 다른 함수입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:takeFive}}
```

첫 번째 패턴은 첫 번째와 두 번째 요소가 각각 0과 1인 경우에만 다섯 개의 요소를 가진 배열과 일치합니다. 이 경우 함수는 세 번째와 네 번째 요소의 곱을 반환합니다. 모든 다른 경우에는 함수가 0을 반환합니다. 예를 들어, PSCi에서:

```text
> :paste
… takeFive [0, 1, a, b, _] = a * b
… takeFive _ = 0
… ^D

> takeFive [0, 1, 2, 3, 4]
6

> takeFive [1, 2, 3, 4, 5]
0

> takeFive []
0
```

배열 리터럴 패턴을 사용하면 고정 길이의 배열과 일치할 수 있습니다. 하지만 PureScript는 불

특정 길이의 배열과 일치하는 방법을 제공하지 않습니다. 이러한 방식으로 불변 배열을 구조분해하면 성능이 저하될 수 있기 때문입니다. 이러한 종류의 매칭을 지원하는 데이터 구조가 필요하다면, `Data.List`를 사용하는 것이 좋습니다. 다른 연산을 위한 비대칭 성능을 제공하는 데이터 구조도 존재합니다.

## 레코드 패턴과 행 다형성

_레코드 패턴_ 은 – 예상했듯이 – 레코드와 일치하는 데 사용됩니다.

레코드 패턴은 레코드 리터럴과 유사하지만, 콜론 오른쪽에 값 대신 바인더를 지정합니다.

예를 들어, 이 패턴은 `first`와 `last`라는 필드를 포함하는 모든 레코드와 일치하며, 해당 값을 각각 `x`와 `y`라는 이름에 바인딩합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:showPerson}}
```

레코드 패턴은 PureScript 타입 시스템의 흥미로운 기능인 _행 다형성_ 의 좋은 예를 제공합니다. 위에서 타입 시그니처 없이 `showPerson`을 정의했다고 가정해 봅시다. 추론된 타입은 어떻게 될까요? 흥미롭게도 우리가 제공한 타입과 동일하지 않습니다:

```text
> showPerson { first: x, last: y } = y <> ", " <> x

> :type showPerson
forall (r :: Row Type). { first :: String, last :: String | r } -> String
```

여기서 타입 변수 `r`은 무엇일까요? PSCi에서 `showPerson`을 시도하면 흥미로운 것을 볼 수 있습니다:

```text
> showPerson { first: "Phil", last: "Freeman" }
"Freeman, Phil"

> showPerson { first: "Phil", last: "Freeman", location: "Los Angeles" }
"Freeman, Phil"
```

레코드에 추가 필드를 추가할 수 있으며, `showPerson` 함수는 여전히 작동합니다. 레코드가 `first`와 `last` 필드를 `String` 타입으로 포함하고 있는 한, 함수 적용은 잘 타입된 것입니다. 그러나 너무 적은 필드를 사용하여 `showPerson`을 호출하는 것은 유효하지 않습니다:

```text
> showPerson { first: "Phil" }

Type of expression lacks required label "last"
```

`showPerson`의 새로운 타입 시그니처를 " `String` 타입의 `first`와 `last` 필드를 포함하고 _다른 모든 필드_ 를 포함하는 레코드를 받아 `String`을 반환하는 함수"로 읽을 수 있습니다. 이 함수는 레코드 필드의 _행_ `r`에 대해 다형성이므로, _행 다형성_ 이라는 이름이 붙었습니다. 원래 `showPerson`의 경우와는 다르게, 행 변수 `r` 없이 `showPerson`은 정확히 `first`와 `last` 필드만을 포함하고 다른 필드가 없는 레코드만을 허용합니다.

다음과 같이 작성할 수도 있습니다:

```haskell
> showPerson p = p.last <> ", " <> p.first
```

그러면 PSCi는 동일한 타입을 추론합니다.

## 레코드 펀

`showPerson` 함수는 인수 내부에서 레코드를 매칭하여 `first`와 `last` 필드를 각각 `x`와 `y`라는 값으로 바인딩합니다. 필드 이름 자체를 재사용하여 이러한 패턴 매칭을 다음과 같이 단순화할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:showPersonV2}}
```

여기서는 필드 이름만 지정하면 되며, 도입하고자 하는 값의 이름을 지정할 필요가 없습니다. 이를 _레코드 펀_ 이라고 합니다.

레코드 펀을 사용하여 레코드를 _구성_ 하는 것도 가능합니다. 예를 들어, 범위 내에 `first`와 `last`라는 값이 있는 경우 `{ first, last }`를 사용하여 사람 레코드를 구성할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:unknownPerson}}
```

이는 특정 상황에서 코드 가독성을 향상시킬 수 있습니다.

## 중첩된 패턴

배열 패턴과 레코드 패턴은 모두 작은 패턴을 결합하여 더 큰 패턴을 만듭니다. 대부분의 예제는 배열 패턴과 레코드 패턴 내에서 단순한 패턴만 사용했습니다. 하지만 패턴은 임의로 _중첩_ 될 수 있으며, 이는 복잡한 데이터 타입에 대한 조건을 사용하여 함수를 정의할 수 있게 합니다.

예를 들어, 이 코드는 두 레코드 패턴을 결합합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:livesInLA}}
```

## 이름 있는 패턴

패턴은 _이름을 붙여_ 중첩 패턴을 사용할 때 추가 이름을 범위 내에 도입할 수 있습니다. 모든 패턴은 `@` 기호를 사용하여 이름을 붙일 수 있습니다.

예를 들어, 이 함수는 두 요소 배열을 정렬하며, 두 요소를 이름 지어 배열 자체에도 이름을 붙입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:sortPair}}
```

이렇게 하면 페어가 이미 정렬되어 있을 경우 새로운 배열을 할당하지 않아도 됩니다. 입력 배열이 정확히 두 개의 요소를 포함하지 않는 경우, 이 함수는 변경되지 않은 상태로 반환됩니다.

## 연습 문제

1. (쉬움) 레코드 패턴을 사용하여 두 `Person` 레코드가 동일한 도시에 속하는지 테스트하는 `sameCity` 함수를 작성하세요.
2. (중간) 행 다형성을 고려할 때 `sameCity` 함수의 가장 일반적인 타입은 무엇인가요? 위에서 정의한 `livesInLA` 함수는 어떨까요? _참고_: 이 연습 문제에는 테스트가 없습니다.
3. (중간) 배열 리터럴 패턴을 사용하여 단일 요소 배열의 유일한 멤버를 추출하는 `fromSingleton` 함수를 작성하세요. 배열이 단일 요소 배열이 아닌 경우, 제공된 기본값을 반환해야 합니다. 함수의 타입은 `forall a. a -> Array a -> a`이어야 합니다.

## 케이스 표현식

패턴은 최상위 함수 선언에만 나타나는 것이 아닙니다. `case` 표현식을 사용하여 계산 중간의 값을 매칭할 수도 있습니다. 케이스 표현식은 익명 함수와 유사한 유틸리티를 제공합니다: 항상 함수에 이름을 붙이는 것이 바람직하지 않으며, `case` 표현식을 사용하면 패턴을 사용하려는 이유만으로 함수를 이름 붙이는 것을 피할 수 있습니다.

다음은 예제입니다. 이 함수는 배열의 "가장 긴 0 접미사" (합이 0인 가장 긴 접미사)를 계산합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:lzsImport}}

{{#include ../exercises/chapter4/src/ChapterExamples.purs:lzs}}
```

예를 들어:

```text
> lzs [1, 2, 3, 4]
[]

> lzs [1, -1, -2, 3]
[-1, -2, 3]
```

이 함수는 케이스 분석을 통해 동작합니다. 배열이 비어 있으면, 빈 배열을 반환하는 것 외에는 선택지가 없습니다. 배열이 비어 있지 않으면, 먼저 `case` 표현식을 사용하여 두 가지 경우로 나눕니다. 배열의 합이 0이면 전체 배열을 반환합니다. 그렇지 않으면, 배열의 꼬리에 대해 재귀 호출을 수행합니다.

## 패턴 매칭 실패와 부분 함수

케이스 표현식의 패턴이 순서대로 시도되면, 어떤 경우에도 입력과 일치하지 않는다면 어떻게 될까요? 이 경우 케이스 표현식은 런타임에서 _패턴 매칭 실패_ 로 실패합니다.

간단한 예제로 이 동작을 확인할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:unsafePartialImport}}

{{#include ../exercises/chapter4/src/ChapterExamples.purs:partialFunction}}
```

이 함수는 단일 입력 `true`와 일치하는 단일 케이스만 포함합니다. 이 파일을 컴파일하고 PSCi에서 다른 인수로 테스트하면 런타임에 오류가 발생합니다:

```text
> partialFunction false

Failed pattern match
```

모든 입력 조합에 대해 값을 반환하는 함수를 _전체_ 함수라고 하고, 그렇지 않은 함수를 _부분_ 함수라고 합니다.

가능한 경우 전체 함수를 정의하는 것이 일반적으로 더 좋습니다. 함수가 일부 유효한 입력 집합에 대해 결과를 반환하지 않는다는 것이 알려진 경우, 일반적으로 `Maybe a`와 같은 타입을 사용하여 실패를 나타낼 수 있는 값을 반환하는 것이 좋습니다. 이를 통해 값의 존재 여부를 타입 안전하게 나타

낼 수 있습니다.

PureScript 컴파일러는 패턴 매칭이 완전하지 않아서 함수가 전체가 아닌 경우 오류를 발생시킵니다. `unsafePartial` 함수를 사용하여 이러한 오류를 무시할 수 있습니다(부분 함수가 안전하다고 확신하는 경우!). 위의 `unsafePartial` 호출을 제거하면, 컴파일러는 다음과 같은 오류를 생성합니다:

```text
A case expression could not be determined to cover all inputs.
The following additional cases are required to cover all inputs:

  false
```

이는 값 `false`가 어떤 패턴에도 일치하지 않음을 알려줍니다. 일반적으로 이러한 경고는 여러 개의 일치하지 않는 케이스를 포함할 수 있습니다.

위에서 타입 시그니처를 생략한 경우:

```haskell
partialFunction true = true
```

PSCi는 특이한 타입을 추론합니다:

```text
> :type partialFunction

Partial => Boolean -> Boolean
```

`=>` 기호를 포함한 타입은 나중에 책에서 더 자세히 다루겠습니다(이는 _타입 클래스_ 와 관련이 있습니다). 현재는 PureScript가 부분 함수를 타입 시스템을 통해 추적하며, 안전할 때 명시적으로 타입 검사기에게 알려야 한다는 것을 알고 있으면 충분합니다.

컴파일러는 특정 경우에 케이스가 _중복_ 인 경우 경고를 생성하기도 합니다(즉, 이전 케이스가 일치할 값을 포함하는 경우):

```haskell
redundantCase :: Boolean -> Boolean
redundantCase true = true
redundantCase false = false
redundantCase false = false
```

이 경우 마지막 케이스가 중복으로 정확히 식별됩니다:

```text
A case expression contains unreachable cases:

  false
```

> _참고_: PSCi는 경고를 표시하지 않으므로, 이 예제를 재현하려면 이 함수를 파일로 저장하고 `spago build`를 사용하여 컴파일해야 합니다.

## 대수 데이터 타입

이 섹션에서는 패턴 매칭과 근본적으로 관련된 PureScript 타입 시스템의 기능인 _대수 데이터 타입_ (또는 _ADTs_)을 소개합니다.

그러나 먼저 이 장의 문제를 해결하는 동기 부여 예제를 고려하겠습니다. 이 예제는 단순한 벡터 그래픽 라이브러리를 구현하는 문제를 해결하기 위한 기초를 제공합니다.

간단한 도형(선, 사각형, 원, 텍스트 등)을 나타내는 타입을 정의하려고 한다고 가정해 봅시다. 객체 지향 언어에서는 아마도 `Shape`라는 인터페이스 또는 추상 클래스를 정의하고, 작업하고자 하는 각 유형의 도형에 대해 하나의 구체적인 하위 클래스를 정의할 것입니다.

그러나 이 접근 방식에는 한 가지 주요 단점이 있습니다: `Shape`를 추상적으로 작업하려면 수행할 수 있는 모든 연산을 식별하고 이를 `Shape` 인터페이스에 정의해야 합니다. 새로운 연산을 추가하는 것이 모듈성을 깨지 않고는 어렵습니다.

대수 데이터 타입은 도형 집합이 사전에 알려진 경우 타입 안전한 방법을 제공합니다. 모듈 방식으로 `Shape`에 새로운 연산을 정의하고 여전히 타입 안전성을 유지할 수 있습니다.

다음은 `Shape`를 대수 데이터 타입으로 나타낸 예입니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:Shape}}

{{#include ../exercises/chapter4/src/Data/Picture.purs:Point}}
```

이 선언은 `Shape`를 서로 다른 생성자의 합으로 정의하고, 각 생성자에 포함된 데이터를 식별합니다. `Shape`는 중심 `Point`와 반지름(숫자)을 포함하는 `Circle`, 또는 `Rectangle`, 또는 `Line`, 또는 `Text` 중 하나입니다. `Shape` 타입의 값을 생성하는 다른 방법은 없습니다.

대수 데이터 타입은 `data` 키워드를 사용하여 도입되며, 뒤에 새로운 타입의 이름과 모든 타입 인수가 옵니다. 타입의 생성자(즉, _데이터 생성자_)는 등호 기호 뒤에 pipe 문자(`|`)로 구분하여 정의됩니다. ADT의 생성자가 포함하는 데이터는 원시 타입에 국한되지 않습니다: 생성자는 레코드, 배열 또는 다른 ADT를 포함할 수 있습니다.

PureScript 표준 라이브러리에서 다른 예제를 살펴보겠습니다. 우리는 이 책의 앞부분에서 선택적 값을 정의하는 데 사용되는 `Maybe` 타입을 보았습니다. 다음은 `maybe` 패키지에서 가져온 정의입니다:

```haskell
data Maybe a = Nothing | Just a
```

이 예제는 타입 매개변수 `a`의 사용을 보여줍니다. 파이프 문자를 "또는"이라는 단어로 읽으면, 정의가 영어처럼 읽힙니다: " `Maybe a` 타입의 값은 `Nothing`이거나, `Just` `a` 타입의 값입니다."

`forall a.` 문법은 데이터 정의에서는 사용되지 않지만, 함수에서는 필요하다는 점을 주의하세요. ADT를 `data` 또는 타입 별칭을 `type`으로 정의할 때는 `forall` 문법이 필요하지 않습니다.

데이터 생성자는 재귀 데이터 구조를 정의하는 데에도 사용할 수 있습니다. 다음은 `a` 타입의 요소로 구성된 단일 연결 리스트를 정의하는 또 다른 예입니다:

```haskell
data List a = Nil | Cons a (List a)
```

이 예제는 `lists` 패키지에서 가져왔습니다. 여기서 `Nil` 생성자는 빈 리스트를 나타내며, `Cons`는 헤드 요소와 꼬리를 사용하여 비어 있지 않은 리스트를 생성하는 데 사용됩니다. 꼬리는 `List a` 데이터 타입을 사용하여 정의되므로, 이는 재귀 데이터 타입입니다.

## ADTs 사용하기

ADT 생성자를 사용하여 값을 구성하는 것은 간단합니다: 함수처럼 적용하고, 해당 생성자가 포함하는 데이터에 해당하는 인수를 제공하면 됩니다.

예를 들어, 위에서 정의한 `Line` 생성자는 두 개의 `Point`를 필요로 하므로, `Line` 생성자를 사용하여 `Shape`를 구성하려면 두 개의 `Point` 타입 인수를 제공해야 합니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:exampleLine}}
```

따라서 대수 데이터 타입의 값을 구성하는 것은 간단하지만, 이를 사용하는 방법은 무엇일까요? 여기에서 패턴 매칭과의 중요한 연결이 나타납니다: 대수 데이터 타입의 값을 소비하는 유일한 방법은 생성자를 매칭하는 패턴을 사용하는 것입니다.

예를 보겠습니다. `Shape`를 `String`으로 변환하려고 한다고 가정해 봅시다. `Shape`를 구성하는 데 사용된 생성자를 발견하기 위해 패턴 매칭을 사용해야 합니다. 이를 다음과 같이 할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:showShape}}

{{#include ../exercises/chapter4/src/Data/Picture.purs:showPoint}}
```

각 생성자는 패턴으로 사용할 수 있으며, 생성자의 인수는 자체 패턴을 사용하여 바인딩할 수 있습니다. `showShape`의 첫 번째 케이스를 고려해 봅시다: `Shape`가 `Circle` 생성자와 일치하면, 두 개의 변수 패턴 `c`와 `r`을 사용하여 `Circle`의 인수(중심 및 반지름)를 범위 내에 가져옵니다. 다른 케이스도 유사합니다.

## 연습 문제

1. (쉬움) 중심이 원점이고 반지름이 10.0인 `Circle` (타입 `Shape`)을 구성하는 `circleAtOrigin` 함수를 작성하세요.
2. (중간) 크기를 2.0배로 늘리고 원점에 중심을 둔 `Shape`를 스케일링하는 `doubleScaleAndCenter` 함수를 작성하세요.
3. (중간) `Shape`에서 텍스트를 추출하는 `shapeText` 함수를 작성하세요. 입력이 `Text`로 구성되지 않은 경우 `Nothing` 생성자를 사용하여 `Maybe String`을 반환해야 합니다.

## Newtypes

_newtypes_ 라는 대수 데이터 타입의 특수한 경우가 있습니다. Newtypes는 `data` 키워드 대신 `newtype` 키워드를 사용하여 도입됩니다.

Newtypes는 _정확히 하나의_ 생성자를 정의해야 하며, 해당 생성자는 _정확히 하나의_ 인수를 가져야 합니다. 즉, newtype은 기존 타입에 새 이름을 부여합니다. 실제로 newtype의 값은 기본 타입과 동일한 런타임 표현을 가지므로 런타임 성능 오버헤드가 없습니다. 그러나 타입 시스템의 관점에서는 별개입니다. 이는 추가적인 타입 안전성을 제공합니다.

예를 들어, 전압, 전류, 저항과 같은 단위를 부여하기 위해 `Number`의 타입 수준 별칭으로 newtype을 정의하고 싶을 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:electricalUnits}}
```

그런 다음 이러한 타입을 사용하여 함수와 값을 정의합니다:

```

haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:calculateCurrent}}
```

이를 통해 두 개의 전구에 전압원이 없이 전류를 계산하려는 실수를 방지할 수 있습니다.

```haskell
current :: Amp
current = calculateCurrent lightbulb lightbulb
{-
TypesDoNotUnify:
  current = calculateCurrent lightbulb lightbulb
                             ^^^^^^^^^
  Could not match type
    Ohm
  with type
    Volt
-}
```

대신 `Number`를 newtype 없이 사용하면, 컴파일러는 이 실수를 도와 잡아내지 않습니다:

```haskell
-- 이는 컴파일되지만, 타입 안전하지 않습니다.
calculateCurrent :: Number -> Number -> Number
calculateCurrent v r = v / r

battery :: Number
battery = 1.5

lightbulb :: Number
lightbulb = 500.0

current :: Number
current = calculateCurrent lightbulb lightbulb -- 잡아내지 못한 실수
```

newtype은 단일 생성자를 가질 수 있으며, 해당 생성자는 단일 값이어야 하지만, newtype은 임의의 수의 타입 변수를 가질 수 있습니다. 예를 들어, 다음 newtype 정의는 유효한 정의입니다(`err`과 `a`는 타입 변수이며, `CouldError` 생성자는 단일 값 `Either err a`를 기대합니다):

```Haskell
newtype CouldError err a = CouldError (Either err a)
```

또한, newtype의 생성자는 종종 newtype 자체와 동일한 이름을 가지지만, 이는 필수 사항이 아닙니다. 예를 들어, 고유한 이름도 유효합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:Coulomb}}
```

이 경우, `Coulomb`은 _타입 생성자_ (인수가 없음)이고, `MakeCoulomb`은 _데이터 생성자_ 입니다. 이러한 생성자는 이름이 동일한 경우에도 다른 네임스페이스에 존재합니다. 이는 모든 ADT에 해당됩니다. 타입 생성자와 데이터 생성자가 다른 이름을 가질 수 있지만, 실제로는 동일한 이름을 공유하는 것이 관례입니다. 이는 위의 `Amp` 및 `Volt` 타입에 해당됩니다.

Newtype의 또 다른 응용은 런타임 표현을 변경하지 않고 기존 타입에 다른 _동작_ 을 첨부하는 것입니다. 다음 장에서는 _타입 클래스_ 를 다룰 때 이러한 사용 사례를 다룹니다.

## 연습 문제

1. (쉬움) `Number`의 `newtype`으로 `Watt`를 정의하세요. 그런 다음, 위의 정의된 `Amp` 및 `Volt` 타입을 사용하여 `Watt` 타입의 `calculateWattage` 함수를 정의하세요:

```haskell
calculateWattage :: Amp -> Volt -> Watt
```

`Watt` 단위의 와트 수는 주어진 `Amp` 단위의 전류와 주어진 `Volt` 단위의 전압의 곱으로 계산할 수 있습니다.

## 벡터 그래픽을 위한 라이브러리

이전에 정의한 데이터 타입을 사용하여 벡터 그래픽을 사용하는 간단한 라이브러리를 만들어 보겠습니다.

`Picture` 타입의 동의어를 정의하세요 – `Shape`의 배열입니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:Picture}}
```

디버깅 목적으로, `Picture`를 읽을 수 있는 형식으로 변환하고 싶습니다. `showPicture` 함수는 이를 가능하게 합니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:showPicture}}
```

시도해 봅시다. `spago build`로 모듈을 컴파일하고 `spago repl`로 PSCi를 엽니다:

```text
$ spago build
$ spago repl

> import Data.Picture

> showPicture [ Line { x: 0.0, y: 0.0 } { x: 1.0, y: 1.0 } ]

["Line [start: (0.0, 0.0), end: (1.0, 1.0)]"]
```

## 경계 사각형 계산

이 모듈의 예제 코드에는 `Picture`의 가장 작은 경계 사각형을 계산하는 `bounds` 함수가 포함되어 있습니다.

`Bounds` 타입은 경계 사각형을 정의합니다.

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:Bounds}}
```

`bounds`는 `Data.Foldable`의 `foldl` 함수를 사용하여 `Picture`의 `Shape` 배열을 순회하고, 가장 작은 경계 사각형을 축적합니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:bounds}}
```

기본 사례에서는 빈 `Picture`의 가장 작은 경계 사각형을 찾아야 하며, `emptyBounds`로 정의된 빈 경계 사각형이 충분합니다.

누적 함수 `combine`은 `where` 블록에서 정의됩니다. `combine`은 `foldl`의 재귀 호출에서 계산된 경계 사각형과 배열의 다음 `Shape`를 받아들이고, 두 경계 사각형의 합집합을 계산하는 `union` 함수를 사용합니다. `shapeBounds` 함수는 패턴 매칭을 사용하여 단일 도형의 경계를 계산합니다.

## 연습 문제

1. (중간) `Shape`의 면적을 계산하는 새 연산 `area`를 벡터 그래픽 라이브러리에 추가하세요. 이 연습 문제의 목적상, 선이나 텍스트의 면적은 0으로 가정합니다.
2. (어려움) `Shape` 타입에 다른 `Picture`를 사각형으로 잘라내는 새로운 데이터 생성자 `Clipped`를 추가하세요. 잘린 그림의 경계를 계산하기 위해 `shapeBounds` 함수를 확장하세요. 이는 `Shape`를 재귀 데이터 타입으로 만듭니다. _힌트_: 컴파일러는 필요한 다른 함수를 확장하는 과정을 안내할 것입니다.

## 결론

이 장에서는 함수형 프로그래밍의 기본적이지만 강력한 기법인 패턴 매칭을 다뤘습니다. 단순한 패턴뿐만 아니라 배열 및 레코드 패턴을 사용하여 깊은 데이터 구조의 일부와 일치시키는 방법을 보았습니다.

이 장에서는 대수 데이터 타입도 소개했으며, 이는 패턴 매칭과 밀접하게 관련되어 있습니다. 대수 데이터 타입이 데이터 구조를 간결하게 설명하고, 새로운 연산으로 데이터 타입을 확장하는 모듈 방식의 방법을 제공하는 방법을 보았습니다.

마지막으로, _행 다형성_ 이라는 강력한 추상화 유형을 다뤘으며, 이를 통해 많은 관습적인 JavaScript 함수를 타입 지정할 수 있게 됩니다.

책의 나머지 부분에서는 ADT와 패턴 매칭을 광범위하게 사용할 예정이므로, 지금 익숙해지면 큰 도움이 될 것입니다. 자신의 대수 데이터 타입을 만들고 패턴 매칭을 사용하여 이를 소비하는 함수를 작성해 보세요.