# 패턴 매칭

> 임시 주의: 이 챕터를 작업 중이라면, 2023년 11월에 챕터 4와 5가 교체되었으니 주의하세요.

## 챕터 목표

이 챕터에서는 두 가지 새로운 개념인 대수적 데이터 타입과 패턴 매칭을 소개합니다. 또한 PureScript 타입 시스템의 흥미로운 기능인 행 다형성(row polymorphism)에 대해서도 간단히 다룹니다.

패턴 매칭은 함수형 프로그래밍에서 흔히 사용되는 기법으로, 개발자가 복잡한 아이디어를 여러 가지 경우로 나누어 간결한 함수를 작성할 수 있게 해줍니다.

대수적 데이터 타입은 타입 언어에서 유사한 수준의 표현력을 가능하게 하는 PureScript 타입 시스템의 기능으로, 패턴 매칭과 밀접한 관련이 있습니다.

이 챕터의 목표는 대수적 타입과 패턴 매칭을 사용하여 간단한 벡터 그래픽을 설명하고 조작하는 라이브러리를 작성하는 것입니다.

## 프로젝트 설정

이 챕터의 소스 코드는 `src/Data/Picture.purs` 파일에 정의되어 있습니다.

`Data.Picture` 모듈은 간단한 도형을 위한 데이터 타입 `Shape`와 도형의 컬렉션을 위한 타입 `Picture`를 정의하며, 해당 타입을 작업하는 함수를 포함합니다.

이 모듈은 데이터 구조를 폴딩하는 함수를 제공하는 `Data.Foldable` 모듈을 임포트합니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:module_picture}}
```

`Data.Picture` 모듈은 이번에는 `as` 키워드를 사용하여 `Number` 모듈도 임포트합니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:picture_import_as}}
```

이는 해당 모듈의 타입과 함수를 사용할 수 있게 하지만, `Number.max`와 같이 **정규화된 이름**을 통해서만 가능합니다. 이는 임포트 간의 겹침을 피하거나 특정 모듈에서 임포트된 것들을 명확히 하는 데 유용합니다.

> **참고**: 정규화된 임포트에 원래 모듈 이름과 동일한 모듈 이름을 사용하는 것은 필수가 아닙니다. `import Data.Number as N`과 같이 더 짧은 정규화된 이름을 사용하는 것이 가능하며 꽤 흔합니다.

## 간단한 패턴 매칭

예제를 보면서 시작해 봅시다. 다음은 패턴 매칭을 사용하여 두 정수의 최대 공약수를 계산하는 함수입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:gcd}}
```

이 알고리즘은 유클리드 알고리즘이라고 합니다. 온라인에서 그 정의를 찾아보면 아마도 위의 코드와 유사한 수학적 방정식을 볼 수 있을 것입니다. 패턴 매칭의 한 가지 이점은 여러 경우로 코드를 정의할 수 있어, 수학적 함수 명세와 유사한 간단하고 선언적인 코드를 작성할 수 있다는 것입니다.

패턴 매칭을 사용하여 작성된 함수는 조건 집합과 그 결과를 짝지음으로써 작동합니다. 각 라인은 **대안(alternative)** 또는 **케이스(case)**라고 불립니다. 등호의 왼쪽에 있는 표현식을 **패턴**이라고 하며, 각 케이스는 공백으로 구분된 하나 이상의 패턴으로 구성됩니다. 케이스는 인수가 어떤 조건을 만족해야 등호의 오른쪽에 있는 표현식을 평가하고 반환해야 하는지를 설명합니다. 각 케이스는 순서대로 시도되며, 패턴이 입력과 일치하는 첫 번째 케이스가 반환 값을 결정합니다.

예를 들어, `gcd` 함수는 다음과 같은 단계로 평가됩니다:

- 첫 번째 케이스 시도: 두 번째 인수가 0이면, 함수는 `n`(첫 번째 인수)을 반환합니다.
- 그렇지 않으면, 두 번째 케이스 시도: 첫 번째 인수가 0이면, 함수는 `m`(두 번째 인수)을 반환합니다.
- 그렇지 않으면, 마지막 라인의 표현식을 평가하고 반환합니다.

패턴은 값에 이름을 바인딩할 수 있다는 점에 유의하세요. 예제의 각 라인은 `n`과 `m`이라는 이름 중 하나 또는 둘 다를 입력 값에 바인딩합니다. 다양한 패턴에 대해 배우면서, 다양한 패턴이 입력 인수에서 이름을 선택하는 다양한 방식에 대응한다는 것을 알게 될 것입니다.

## 간단한 패턴

위의 예제 코드는 두 가지 유형의 패턴을 보여줍니다:

- 정수 리터럴 패턴: 값이 정확히 일치하는 경우에만 `Int` 타입의 값을 매치합니다.
- 변수 패턴: 인수를 이름에 바인딩합니다.

다른 유형의 간단한 패턴도 있습니다:

- `Number`, `String`, `Char`, `Boolean` 리터럴
- 밑줄(`_`)로 표시되는 와일드카드 패턴은 어떤 인수와도 매치하며 어떤 이름에도 바인딩하지 않습니다.

다음은 이러한 간단한 패턴을 사용하는 두 가지 추가 예제입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:fromString}}

{{#include ../exercises/chapter4/src/ChapterExamples.purs:toString}}
```

PSCi에서 이 함수들을 시도해 보세요.

## 가드(Guards)

유클리드 알고리즘 예제에서 `m > n`과 `m <= n`일 때 두 가지 대안을 전환하기 위해 `if .. then .. else` 표현식을 사용했습니다. 이 경우, **가드**를 사용하는 또 다른 옵션이 있습니다.

가드는 패턴이 부과하는 제약 조건 외에도 만족해야 하는 부울 값의 표현식입니다. 다음은 가드를 사용하여 다시 작성된 유클리드 알고리즘입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:gcdV2}}
```

이 경우, 세 번째 라인은 첫 번째 인수가 두 번째 인수보다 엄격히 크다는 추가 조건을 부과하기 위해 가드를 사용합니다. 마지막 라인의 가드는 표현식 `otherwise`를 사용하는데, 이는 키워드처럼 보이지만 실제로는 `Prelude`에 있는 일반 바인딩입니다:

```text
> :type otherwise
Boolean

> otherwise
true
```

이 예제는 가드가 패턴 목록과 등호 사이에 파이프 문자(`|`)로 구분되어 등호의 왼쪽에 나타난다는 것을 보여줍니다.

## 연습문제

1. (쉬움) 패턴 매칭을 사용하여 `factorial` 함수를 작성하세요. **힌트**: 0과 0이 아닌 입력의 두 가지 코너 케이스를 고려하세요. **참고**: 이전 챕터의 예제와 중복되지만, 여기서 직접 다시 작성해 보세요.
2. (중간) 다항식 전개 \\( ( 1 + x ) ^ n \\)에서 \\( x ^ k \\)항의 계수를 찾는 `binomial` 함수를 작성하세요. 이는 \\( n \\)개의 요소 중에서 \\( k \\)개의 부분 집합을 선택하는 방법의 수와 같습니다. \\( ! \\)가 이전에 작성한 팩토리얼 함수인 공식을 사용하세요: \\( n! \\: / \\: k! \\, (n - k)! \\). **힌트**: 코너 케이스를 처리하기 위해 패턴 매칭을 사용하세요. 완료하는 데 시간이 오래 걸리거나 호출 스택에 대한 오류로 충돌하는 경우, 더 많은 코너 케이스를 추가해 보세요.
3. (중간) 동일한 이항 계수를 계산하기 위해 [**파스칼의 법칙**](https://en.wikipedia.org/wiki/Pascal%27s_rule)을 사용하는 `pascal` 함수를 작성하세요.

## 배열 패턴

**배열 리터럴 패턴**은 고정 길이의 배열을 매치하는 방법을 제공합니다. 예를 들어, 빈 배열을 식별하는 `isEmpty` 함수를 작성하고 싶다고 가정해 봅시다. 첫 번째 대안에서 빈 배열 패턴(`[]`)을 사용하여 이를 수행할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:isEmpty}}
```

다음은 다섯 개의 요소를 가진 배열을 매치하고 각 요소를 다르게 바인딩하는 또 다른 함수입니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:takeFive}}
```

첫 번째 패턴은 첫 번째와 두 번째 요소가 각각 0과 1인 다섯 개의 요소를 가진 배열과만 매치됩니다. 이 경우, 함수는 세 번째와 네 번째 요소의 곱을 반환합니다. 다른 모든 경우에는 함수는 0을 반환합니다. 예를 들어, PSCi에서:

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

배열 리터럴 패턴을 사용하면 고정 길이의 배열을 매치할 수 있습니다. 그러나 PureScript는 **불특정 길이**의 배열을 매치하는 수단을 제공하지 않습니다. 이는 이러한 방식으로 불변 배열을 구조 분해하면 성능이 저하될 수 있기 때문입니다. 이러한 종류의 매칭을 지원하는 데이터 구조가 필요한 경우, 권장되는 접근 방식은 `Data.List`를 사용하는 것입니다. 다른 데이터 구조도 다양한 작업에 대한 점근적 성능을 개선하기 위해 존재합니다.

## 레코드 패턴과 행 다형성

**레코드 패턴**은 레코드를 매치하는 데 사용됩니다.

레코드 패턴은 레코드 리터럴과 거의 동일하게 보이지만, 콜론의 오른쪽에 값 대신 각 필드에 대한 바인더를 지정합니다.

예를 들어, 이 패턴은 `first`와 `last`라는 필드를 포함하는 모든 레코드와 매치하며, 그 값을 각각 `x`와 `y`라는 이름에 바인딩합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:showPerson}}
```

레코드 패턴은 PureScript 타입 시스템의 흥미로운 기능인 **행 다형성(row polymorphism)**의 좋은 예를 제공합니다. 위에서 타입 시그니처 없이 `showPerson`을 정의했다고 가정해 봅시다. 그럼 추론된 타입은 무엇일까요? 흥미롭게도 우리가 제공한 타입과 동일하지 않습니다:

```text
> showPerson { first: x, last: y } = y <> ", " <> x

> :type showPerson
forall (r :: Row Type). { first :: String, last :: String | r } -> String
```

여기서 타입 변수 `r`은 무엇일까요? PSCi에서 `showPerson`을 시도해 보면 흥미로운 것을 볼 수 있습니다:

```text
> showPerson { first: "Phil", last: "Freeman" }
"Freeman, Phil"

> showPerson { first: "Phil", last: "Freeman", location: "Los Angeles" }
"Freeman, Phil"
```

레코드에 추가 필드를 추가해도 `showPerson` 함수는 여전히 작동합니다. `first`와 `last` 필드가 `String` 타입으로 레코드에 포함되어 있는 한, 함수 적용은 타입이 맞습니다. 그러나 **너무 적은** 필드로 `showPerson`을 호출하는 것은 유효하지 않습니다:

```text
> showPerson { first: "Phil" }

Type of expression lacks required label "last"
```

`showPerson`의 새로운 타입 시그니처를 "추가적인 **어떤** 필드를 가진 `String` 타입의 `first`와 `last` 필드를 가진 **어떤** 레코드를 받아서 `String`을 반환한다"고 읽을 수 있습니다. 이 함수는 레코드 필드의 **행** `r`에 대해 다형적이므로, **행 다형성**이라는 이름이 붙었습니다. 이 동작은 원래의 `showPerson`과는 다릅니다. `r`이라는 행 변수가 없으면, `showPerson`은 정확히 `first`와 `last` 필드만 있고 다른 필드가 없는 레코드만 허용합니다.

또한 다음과 같이 작성할 수도 있습니다:

```haskell
> showPerson p = p.last <> ", " <> p.first
```

PSCi는 동일한 타입을 추론합니다.

## 레코드 펀(Record Puns)

`showPerson` 함수는 인수 내부에서 레코드를 매치하여 `first`와 `last` 필드를 `x`와 `y`라는 값에 바인딩합니다. 필드 이름 자체를 재사용하고 이러한 종류의 패턴 매치를 다음과 같이 단순화할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:showPersonV2}}
```

여기서는 필드 이름만 지정하면 되며, 도입하려는 값의 이름을 지정할 필요가 없습니다. 이를 **레코드 펀(record pun)**이라고 합니다.

또한 레코드 펀을 사용하여 레코드를 **구성**할 수도 있습니다. 예를 들어, 스코프 내에 `first`와 `last`라는 값이 있으면 `{ first, last }`를 사용하여 사람 레코드를 구성할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:unknownPerson}}
```

이는 특정 상황에서 코드의 가독성을 향상시킬 수 있습니다.

## 중첩 패턴

배열 패턴과 레코드 패턴은 모두 작은 패턴을 결합하여 더 큰 패턴을 만듭니다. 대부분 위의 예제들은 배열 패턴과 레코드 패턴 내에서 간단한 패턴만 사용했습니다. 그러나 패턴은 임의로 **중첩**될 수 있으며, 이는 복잡한 데이터 타입의 부분에 조건을 사용하여 함수를 정의할 수 있게 해줍니다.

예를 들어, 다음 코드는 두 개의 레코드 패턴을 결합합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:livesInLA}}
```

## 이름 붙은 패턴

패턴은 **이름을 붙일 수 있으며**, 중첩 패턴을 사용할 때 추가적인 이름을 스코프 내로 가져올 수 있습니다. 모든 패턴은 `@` 기호를 사용하여 이름을 붙일 수 있습니다.

예를 들어, 이 함수는 두 요소 배열을 정렬하며, 두 요소의 이름을 지정하면서 배열 자체에도 이름을 지정합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:sortPair}}
```

이렇게 하면 쌍이 이미 정렬된 경우 새로운 배열을 할당하는 것을 피할 수 있습니다. 입력 배열이 **정확히** 두 요소를 포함하지 않는 경우, 이 함수는 정렬되지 않은 경우에도 변경 없이 반환합니다.

## 연습문제

1. (쉬움) 두 `Person` 레코드가 동일한 도시에 속하는지 테스트하기 위해 레코드 패턴을 사용하는 `sameCity` 함수를 작성하세요.
2. (중간) 행 다형성을 고려하여 `sameCity` 함수의 가장 일반적인 타입은 무엇인가요? 위에서 정의한 `livesInLA` 함수는 어떤가요? **참고**: 이 연습문제에는 테스트가 없습니다.
3. (중간) 배열 리터럴 패턴을 사용하여 단일 요소 배열의 유일한 멤버를 추출하는 `fromSingleton` 함수를 작성하세요. 배열이 단일 요소가 아닌 경우, 함수는 제공된 기본 값을 반환해야 합니다. 함수의 타입은 `forall a. a -> Array a -> a`이어야 합니다.

## 케이스 표현식

패턴은 최상위 함수 선언에서만 나타나는 것이 아닙니다. 패턴을 사용하여 중간 값에서 매치하기 위해 `case` 표현식을 사용할 수 있습니다. 케이스 표현식은 익명 함수와 유사한 유틸리티를 제공합니다: 항상 함수에 이름을 지정하는 것이 바람직한 것은 아니며, `case` 표현식을 사용하면 패턴을 사용하기 위해 함수를 이름 짓는 것을 피할 수 있습니다.

예를 들어 보겠습니다. 이 함수는 배열의 "가장 긴 0 접미사"(합이 0인 가장 긴 접미사)를 계산합니다:

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

이 함수는 케이스 분석을 통해 작동합니다. 배열이 비어 있으면, 우리가 할 수 있는 유일한 선택은 빈 배열을 반환하는 것입니다. 배열이 비어 있지 않으면 먼저 `case` 표현식을 사용하여 두 가지 경우로 나눕니다. 배열의 합이 0이면 전체 배열을 반환합니다. 그렇지 않으면 배열의 꼬리에 재귀적으로 적용합니다.

## 패턴 매치 실패와 부분 함수

케이스 표현식에서 케이스 대안의 패턴이 순서대로 시도된다면, 어떤 패턴도 입력과 매치되지 않으면 어떻게 될까요? 이 경우, 케이스 표현식은 런타임에 **패턴 매치 실패**로 실패합니다.

간단한 예제로 이 동작을 볼 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:unsafePartialImport}}

{{#include ../exercises/chapter4/src/ChapterExamples.purs:partialFunction}}
```

이 함수는 단일 케이스만 포함하며, 이는 단일 입력 `true`와만 매치됩니다. 이 파일을 컴파일하고 다른 인수로 PSCi에서 테스트하면, 런타임에 오류가 발생합니다:

```text
> partialFunction false

Failed pattern match
```

모든 입력 조합에 대해 값을 반환하는 함수를 **전역 함수(total function)**라고 하며, 그렇지 않은 함수를 **부분 함수(partial function)**라고 합니다.

가능하면 전역 함수를 정의하는 것이 일반적으로 더 좋다고 여겨집니다. 함수가 일부 유효한 입력 세트에 대해 결과를 반환하지 않는다는 것이 알려진 경우, `Maybe a` 타입과 같이 실패를 나타낼 수 있는 값을 반환하는 것이 보통 더 좋습니다. 이렇게 하면 값의 존재 여부를 타입 안전한 방식으로 나타낼 수 있습니다.

PureScript 컴파일러는 패턴 매칭이 불완전하여 함수가 전역적이지 않음을 감지할 수 있으면 오류를 생성합니다. 위에서 `unsafePartial` 함수를 호출하는 것을 제거하면, 컴파일러는 다음과 같은 오류를 생성합니다:

```text
A case expression could not be determined to cover all inputs.
The following additional cases are required to cover all inputs:

  false
```

이는 값 `false`가 어떤 패턴에도 매치되지 않음을 알려줍니다. 일반적으로 이러한 경고에는 매치되지 않은 여러 케이스가 포함될 수 있습니다.

위에서 타입 시그니처도 생략하면:

```haskell
partialFunction true = true
```

PSCi는 흥미로운 타입을 추론합니다:

```text
> :type partialFunction

Partial => Boolean -> Boolean
```

`=>` 기호가 포함된 타입은 다음 장에서 더 많이 다루겠지만(이는 **타입 클래스**와 관련이 있습니다), 지금은 PureScript가 부분 함수를 타입 시스템을 사용하여 추적하며, 우리가 명시적으로 타입 검사기에 안전하다고 알려주어야 한다는 것을 이해하면 됩니다.

컴파일러는 또한 케이스가 **중복**됨을 감지할 수 있는 경우(즉, 케이스가 이전 케이스가 매치할 값만 매치하는 경우) 경고를 생성합니다:

```haskell
redundantCase :: Boolean -> Boolean
redundantCase true = true
redundantCase false = false
redundantCase false = false
```

이 경우, 마지막 케이스는 중복으로 정확히 식별됩니다:

```text
A case expression contains unreachable cases:

  false
```

> **참고**: PSCi는 경고를 표시하지 않으므로, 이 예제를 재현하려면 함수를 파일로 저장하고 `spago build`를 사용하여 컴파일해야 합니다.

## 대수적 데이터 타입

이 섹션에서는 패턴 매칭과 근본적으로 관련된 PureScript 타입 시스템의 기능인 **대수적 데이터 타입(Algebraic Data Types, ADTs)**를 소개합니다.

그러나 먼저 동기 부여 예제를 고려해 보겠습니다. 이는 이 챕터의 문제인 간단한 벡터 그래픽 라이브러리를 구현하는 솔루션의 기반을 제공합니다.

간단한 도형(선, 사각형, 원, 텍스트 등)을 나타내는 타입을 정의하고 싶다고 가정해 봅시다. 객체 지향 언어에서는 아마도 인터페이스나 추상 클래스 `Shape`를 정의하고, 작업하려는 각 유형의 도형에 대해 하나의 구체적인 서브클래스를 정의할 것입니다.

그러나 이 접근 방식에는 한 가지 큰 단점이 있습니다: `Shape`를 추상적으로 작업하려면 수행하려는 모든 작업을 식별하고 이를 `Shape` 인터페이스에 정의해야 합니다. 새로운 작업을 추가하는 것이 모듈성을 깨지 않고는 어렵습니다.

대수적 데이터 타입은 도형의 집합이 사전에 알려져 있는 경우 타입 안전한 방식으로 이 문제를 해결할 수 있습니다. 데이터 타입에 새로운 작업을 모듈 방식으로 정의할 수 있으며, 여전히 타입 안전성을 유지할 수 있습니다.

다음은 대수적 데이터 타입으로 `Shape`를 나타내는 방법입니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:Shape}}

{{#include ../exercises/chapter4/src/Data/Picture.purs:Point}}
```

이 선언은 `Shape`를 다양한 생성자의 합으로 정의하며, 각 생성자에 포함된 데이터를 식별합니다. `Shape`는 중심 `Point`와 반지름(숫자)을 포함하는 `Circle`이거나, `Rectangle`, `Line`, `Text`일 수 있습니다. `Shape` 타입의 값을 생성하는 다른 방법은 없습니다.

대수적 데이터 타입은 `data` 키워드를 사용하여 도입되며, 새로운 타입의 이름과 필요한 타입 인수를 따릅니다. 타입의 생성자(즉, **데이터 생성자**)는 등호 뒤에 정의되며, 파이프 문자(`|`)로 구분됩니다. ADT의 생성자가 포함하는 데이터는 원시 타입으로 제한될 필요가 없으며, 생성자는 레코드, 배열 또는 다른 ADT도 포함할 수 있습니다.

PureScript 표준 라이브러리의 또 다른 예제를 봅시다. 이전에 `Maybe` 타입을 보았으며, 이는 선택적 값을 정의하는 데 사용됩니다. 다음은 `maybe` 패키지의 정의입니다:

```haskell
data Maybe a = Nothing | Just a
```

이 예제는 타입 매개변수 `a`의 사용을 보여줍니다. 파이프 문자를 "또는"이라는 단어로 읽으면, 정의는 거의 영어처럼 읽힙니다: "`Maybe a` 타입의 값은 `Nothing`이거나, `a` 타입의 값을 `Just`한 것입니다."

`forall a.` 구문을 데이터 정의 어디에도 사용하지 않는다는 점에 유의하세요. `forall` 구문은 함수에 필요하지만, `data`로 ADT를 정의하거나 `type`으로 타입 별칭을 정의할 때는 사용되지 않습니다.

데이터 생성자는 재귀적인 데이터 구조를 정의하는 데에도 사용할 수 있습니다. 다음은 요소 타입 `a`의 단일 연결 리스트의 데이터 타입을 정의하는 또 다른 예제입니다:

```haskell
data List a = Nil | Cons a (List a)
```

이 예제는 `lists` 패키지에서 가져왔습니다. 여기서 `Nil` 생성자는 빈 리스트를 나타내며, `Cons`는 머리 요소와 꼬리를 사용하여 비어 있지 않은 리스트를 생성하는 데 사용됩니다. 꼬리는 데이터 타입 `List a`를 사용하여 정의되므로, 이는 재귀적인 데이터 타입입니다.

## ADT 사용하기

대수적 데이터 타입의 생성자를 사용하여 값을 구성하는 것은 간단합니다: 해당 생성자를 함수처럼 적용하고, 적절한 생성자와 함께 포함된 데이터에 해당하는 인수를 제공합니다.

예를 들어, 위에서 정의한 `Line` 생성자는 두 개의 `Point`를 필요로 하므로, `Line` 생성자를 사용하여 `Shape`를 구성하려면 `Point` 타입의 두 인수를 제공해야 합니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:exampleLine}}
```

따라서 대수적 데이터 타입의 값을 구성하는 것은 간단하지만, 이를 어떻게 사용할까요? 여기서 패턴 매칭과의 중요한 연결이 나타납니다: 대수적 데이터 타입의 값을 소비하는 유일한 방법은 생성자에 매치하는 패턴을 사용하는 것입니다.

예제를 보겠습니다. `Shape`를 `String`으로 변환하고 싶다고 가정해 봅시다. 우리는 `Shape`를 구성하는 데 사용된 생성자를 알아내기 위해 패턴 매칭을 사용해야 합니다. 다음과 같이 할 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:showShape}}

{{#include ../exercises/chapter4/src/Data/Picture.purs:showPoint}}
```

각 생성자는 패턴으로 사용될 수 있으며, 생성자의 인수는 자체 패턴을 사용하여 바인딩될 수 있습니다. `showShape`의 첫 번째 케이스를 고려해 봅시다: `Shape`가 `Circle` 생성자와 매치되면, 두 개의 변수 패턴 `c`와 `r`을 사용하여 `Circle`의 인수(중심과 반지름)를 스코프로 가져옵니다. 다른 케이스도 유사합니다.

## 연습문제

1. (쉬움) 중심이 원점이고 반지름이 `10.0`인 `Circle`(`Shape` 타입)을 구성하는 `circleAtOrigin` 함수를 작성하세요.
2. (중간) `Shape`의 크기를 `2.0` 배로 스케일링하고 원점에 중심을 맞추는 `doubleScaleAndCenter` 함수를 작성하세요.
3. (중간) `Shape`에서 텍스트를 추출하는 `shapeText` 함수를 작성하세요. 이는 `Maybe String`을 반환해야 하며, 입력이 `Text`를 사용하여 구성되지 않은 경우 `Nothing` 생성자를 사용해야 합니다.

## 뉴타입(Newtypes)

대수적 데이터 타입의 특별한 경우로 **뉴타입(newtype)**이 있습니다. 뉴타입은 `data` 키워드 대신 `newtype` 키워드를 사용하여 도입됩니다.

뉴타입은 **정확히 하나의** 생성자를 정의해야 하며, 그 생성자는 **정확히 하나의** 인수를 가져야 합니다. 즉, 뉴타입은 기존 타입에 새로운 이름을 부여합니다. 실제로, 뉴타입의 값은 기본 타입과 동일한 런타임 표현을 가지므로, 런타임 성능 오버헤드가 없습니다. 그러나 타입 시스템의 관점에서 분리되어 있습니다. 이는 추가적인 타입 안전성 계층을 제공합니다.

예를 들어, 볼트, 암페어, 옴과 같은 단위를 부여하기 위해 `Number`에 대한 타입 수준의 별칭으로 뉴타입을 정의하고 싶을 수 있습니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:electricalUnits}}
```

그런 다음 이러한 타입을 사용하여 함수와 값을 정의합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:calculateCurrent}}
```

이는 우리가 **전압원 없이** 두 개의 전구로부터 전류를 계산하려는 것과 같은 어리석은 실수를 방지해 줍니다.

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

대신에 `newtype` 없이 `Number`만 사용하면, 컴파일러는 이러한 실수를 잡아낼 수 없습니다:

```haskell
-- 이는 컴파일되지만, 타입 안전성이 떨어집니다.
calculateCurrent :: Number -> Number -> Number
calculateCurrent v r = v / r

battery :: Number
battery = 1.5

lightbulb :: Number
lightbulb = 500.0

current :: Number
current = calculateCurrent lightbulb lightbulb -- 잡히지 않는 실수
```

뉴타입은 단일 생성자만 가질 수 있으며, 생성자는 단일 값이어야 하지만, 뉴타입은 **임의의 수의 타입 변수**를 가질 수 있다는 점에 유의하세요. 예를 들어, 다음 뉴타입은 유효한 정의입니다(`err`와 `a`는 타입 변수이며, `CouldError` 생성자는 타입 `Either err a`의 **단일** 값을 기대합니다):

```Haskell
newtype CouldError err a = CouldError (Either err a)
```

또한 뉴타입의 생성자는 종종 뉴타입 자체와 동일한 이름을 가지지만, 이는 필수가 아닙니다. 예를 들어, 고유한 이름도 유효합니다:

```haskell
{{#include ../exercises/chapter4/src/ChapterExamples.purs:Coulomb}}
```

이 경우, `Coulomb`은 **타입 생성자**(인수가 없는)이고, `MakeCoulomb`은 **데이터 생성자**입니다. 이러한 생성자는 서로 다른 네임스페이스에 존재하며, `Volt` 예제와 같이 이름이 동일한 경우에도 그렇습니다. 이는 모든 ADT에 해당됩니다. 타입 생성자와 데이터 생성자는 다른 이름을 가질 수 있지만, 실제로는 동일한 이름을 공유하는 것이 관례입니다. 이는 위의 `Amp`와 `Volt` 타입에서 그렇습니다.

뉴타입의 또 다른 응용은 런타임에서 표현을 변경하지 않고 기존 타입에 다른 **동작**을 부여하는 것입니다. 이 사용 사례는 다음 장에서 **타입 클래스**를 논의할 때 다룹니다.

## 연습문제

1. (쉬움) `Number`의 `newtype`으로 `Watt`를 정의하세요. 그런 다음 위의 `Amp`와 `Volt` 정의를 사용하여 이 새로운 `Watt` 타입을 사용하는 `calculateWattage` 함수를 정의하세요:

```haskell
calculateWattage :: Amp -> Volt -> Watt
```

`Watt` 단위의 와트 수는 주어진 `Amp` 단위의 전류와 주어진 `Volt` 단위의 전압의 곱으로 계산할 수 있습니다.

## 벡터 그래픽을 위한 라이브러리

위에서 정의한 데이터 타입을 사용하여 벡터 그래픽을 사용하는 간단한 라이브러리를 만들어 봅시다.

`Picture`에 대한 타입 별칭을 정의합니다 – 이는 `Shape`의 배열입니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:Picture}}
```

디버깅 목적으로, `Picture`를 읽을 수 있는 것으로 변환하고 싶습니다. `showPicture` 함수가 이를 가능하게 해줍니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:showPicture}}
```

한번 시도해 봅시다. `spago build`로 모듈을 컴파일하고 `spago repl`로 PSCi를 엽니다:

```text
$ spago build
$ spago repl

> import Data.Picture

> showPicture [ Line { x: 0.0, y: 0.0 } { x: 1.0, y: 1.0 } ]

["Line [start: (0.0, 0.0), end: (1.0, 1.0)]"]
```

## 경계 사각형 계산하기

이 모듈의 예제 코드는 `Picture`의 가장 작은 경계 사각형을 계산하는 `bounds` 함수를 포함합니다.

`Bounds` 타입은 경계 사각형을 정의합니다.

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:Bounds}}
```

`bounds`는 `Data.Foldable`의 `foldl` 함수를 사용하여 `Picture`의 `Shape` 배열을 순회하고, 가장 작은 경계 사각형을 누적합니다:

```haskell
{{#include ../exercises/chapter4/src/Data/Picture.purs:bounds}}
```

기본 케이스에서는 빈 `Picture`의 가장 작은 경계 사각형을 찾아야 하며, `emptyBounds`로 정의된 빈 경계 사각형이 충분합니다.

누적 함수 `combine`은 `where` 블록에서 정의됩니다. `combine`은 `foldl`의 재귀 호출에서 계산된 경계 사각형과 배열의 다음 `Shape`를 받아 두 경계 사각형의 합집합을 계산하기 위해 `union` 함수를 사용합니다. `shapeBounds` 함수는 패턴 매칭을 사용하여 단일 도형의 경계를 계산합니다.

## 연습문제

1. (중간) 벡터 그래픽 라이브러리를 확장하여 `Shape`의 면적을 계산하는 새로운 연산 `area`를 추가하세요. 이 연습을 위해, 선이나 텍스트 조각의 면적은 0으로 가정합니다.
2. (어려움) `Shape` 타입에 새로운 데이터 생성자 `Clipped`를 추가하여 다른 `Picture`를 사각형에 클리핑합니다. 클리핑된 그림의 경계를 계산하기 위해 `shapeBounds` 함수를 확장하세요. 이는 `Shape`를 재귀적인 데이터 타입으로 만듭니다. **힌트**: 컴파일러가 필요한 대로 다른 함수를 확장하도록 안내할 것입니다.

## 결론

이 챕터에서는 함수형 프로그래밍의 기본이지만 강력한 기술인 패턴 매칭을 다루었습니다. 간단한 패턴뿐만 아니라 배열과 레코드 패턴을 사용하여 깊은 데이터 구조의 일부를 매치하는 방법을 보았습니다.

또한 패턴 매칭과 밀접한 관련이 있는 대수적 데이터 타입을 소개했습니다. 대수적 데이터 타입이 데이터 구조를 간결하게 설명하고, 새로운 연산으로 데이터 타입을 모듈 방식으로 확장할 수 있는 방법을 보았습니다.

마지막으로 많은 관용적인 JavaScript 함수에 타입을 부여할 수 있는 강력한 추상화인 **행 다형성**을 다루었습니다.

책의 나머지 부분에서 ADT와 패턴 매칭을 광범위하게 사용할 것이므로, 지금 익숙해지는 것이 도움이 될 것입니다. 자신만의 대수적 데이터 타입을 만들고 패턴 매칭을 사용하여 이를 소비하는 함수를 작성해 보세요.
