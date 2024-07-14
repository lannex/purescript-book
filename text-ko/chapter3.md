# 함수와 레코드

## 챕터 목표

이번 챕터에서는 PureScript 프로그램의 두 가지 기본 구성 요소인 함수와 레코드를 소개합니다.
또한 PureScript 프로그램의 구조화 방법과 타입을 프로그램 개발의 보조 도구로 사용하는 방법을 배울 것입니다.

우리는 주소록 애플리케이션을 만들어 연락처 목록을 관리할 것입니다.
이 코드는 PureScript 문법에서 몇 가지 새로운 아이디어를 소개할 것입니다.

애플리케이션의 프런트엔드는 인터랙티브 모드 PSCi가 될 것이지만, 이 코드를 기반으로 JavaScript로 프런트엔드를 작성할 수도 있습니다.
실제로 이후 챕터에서 폼 유효성 검사와 저장/복원 기능을 추가하여 그렇게 할 것입니다.

## 프로젝트 설정

이 챕터의 소스 코드는 `src/Data/AddressBook.purs` 파일에 포함되어 있습니다.
이 파일은 모듈 선언과 그 임포트 리스트로 시작합니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:imports}}
```

여기서 여러 모듈을 가져옵니다:

- `Prelude` 모듈: 소수의 표준 정의와 함수들을 포함하며, `purescript-prelude` 라이브러리에서 많은 기초 모듈을 다시 내보냅니다.
- `Control.Plus` 모듈: `empty` 값을 정의합니다.
- `Data.List` 모듈: Spago를 통해 설치할 수 있는 `lists` 패키지에서 제공되며, 연결 리스트를 작업하기 위해 필요한 몇 가지 함수가 포함되어 있습니다.
- `Data.Maybe` 모듈: 옵셔널 값들을 작업하기 위한 데이터 타입과 함수를 정의합니다.

이 모듈들의 가져오기는 명시적으로 괄호 안에 나열되어 있습니다 (`Prelude`를 제외하고, 이는 일반적으로 열린 임포트로 가져옵니다).
이는 가져오기할 때 충돌을 피하는 데 도움이 되므로 일반적으로 좋은 관행입니다.

책의 소스 코드 레포를 클론했다고 가정하면, 다음 명령어를 사용하여 Spago로 이 챕터의 프로젝트를 빌드할 수 있습니다:

```text
$ cd chapter3
$ spago build
```

## 간단한 타입

PureScript는 JavaScript의 원시 타입에 해당하는 세 가지 내장 타입을 정의합니다: 숫자, 문자열, 불리언.
이들은 `Prim` 모듈에 정의되어 있으며, 각 모듈에 암시적으로 가져옵니다.
각각 `Number`, `String`, `Boolean`이라고 불리며, 몇 가지 간단한 값의 타입을 출력하기 위해 `:type` 명령을 사용하여 PSCi에서 확인할 수 있습니다:

```text
$ spago repl

> :type 1.0
Number

> :type "test"
String

> :type true
Boolean
```

PureScript는 정수, 문자, 배열, 레코드 및 함수와 같은 다른 내장 타입도 정의합니다.

정수는 `Number` 타입의 부동 소수점 값과는 달리 소수점이 없습니다:

```text
> :type 1
Int
```

문자 리터럴은 문자열 리터럴이 큰따옴표를 사용하는 것과 달리 작은따옴표로 감싸져 있습니다:

```text
> :type 'a'
Char
```

배열은 JavaScript의 배열에 해당하지만, JavaScript와 달리 PureScript 배열의 모든 요소는 동일한 타입을 가져야 합니다:

```text
> :type [1, 2, 3]
Array Int

> :type [true, false]
Array Boolean

> :type [1, false]
Could not match type Int with type Boolean.
```

마지막 예시는 두 요소의 타입을 _통합_ (즉, 동일하게 만들기)하지 못한 타입 검사기의 오류를 보여줍니다.

레코드는 JavaScript의 객체에 해당하며, 레코드 리터럴은 JavaScript의 객체 리터럴과 동일한 문법을 가집니다:

```text
> author = { name: "Phil", interests: ["Functional Programming", "JavaScript"] }

> :type author
{ name :: String
, interests :: Array String
}
```

이 타입은 지정된 객체에 두 개의 _필드_ 가 있음을 나타냅니다: `name` 필드는 `String` 타입을 가지며, `interests` 필드는 `Array String` 타입을 가집니다, 즉, `String`의 배열입니다.

레코드의 필드는 점을 사용하여 접근할 수 있습니다:

```text
> author.name
"Phil"

> author.interests
["Functional Programming","JavaScript"]
```

PureScript의 함수는 JavaScript의 함수에 해당합니다.
함수는 파일의 최상위에서 인수를 이퀄 사인 앞에 지정하여 정의할 수 있습니다:

```haskell
import Prelude -- (+) 연산자를 가져옴

add :: Int -> Int -> Int
add x y = x + y
```

또는 백슬래시 문자 뒤에 공백으로 구분된 인수 이름 목록을 사용하는 인라인 함수로 정의할 수 있습니다.
여러 줄로 된 선언을 PSCi에 입력하려면 `:paste` 명령을 사용하여 "붙여넣기 모드"로 들어갈 수 있습니다.
이 모드에서는 _Control-D_ 키 시퀀스를 사용하여 선언을 종료합니다:

```text
> import Prelude
> :paste
… add :: Int -> Int -> Int
… add = \x y -> x + y
… ^D
```

이 함수를 PSCi에 정의한 후, 함수 이름 뒤에 두 인수를 공백으로 구분하여 _적용_ 할 수 있습니다:

```text
> add 10 20
30
```

## 들여쓰기 주의사항

PureScript 코드는 JavaScript와 달리 Haskell과 같이 _들여쓰기에 민감_ 합니다.
이는 코드의 공백이 의미 없지 않으며, C 계열 언어의 중괄호처럼 코드 영역을 그룹화하는 데 사용됨을 의미합니다.

선언이 여러 줄에 걸쳐 있으면 첫 줄을 제외한 나머지 줄은 첫 줄의 들여쓰기 수준을 초과하여 들여쓰기 되어야 합니다.

따라서 다음은 유효한 PureScript 코드입니다:

```haskell
add x y z = x +
  y + z
```

그러나 이것은 유효한 코드가 아닙니다:

```haskell
add x y z = x +
y + z
```

두 번째 경우에서 PureScript 컴파일러는 각 줄에 대해 _두 개_ 의 선언을 구문 분석하려고 시도합니다.

일반적으로 동일한 블록에 정의된 모든 선언은 동일한 수준으로 들여쓰기 되어야 합니다.
예를 들어, PSCi에서 `let` 문에 있는 선언은 동일한 수준으로 들여쓰기 되어야 합니다.
이는 유효합니다:

```text
> :paste
… x = 1
… y = 2
… ^D
```

그러나 이것은 유효하지 않습니다:

```text
> :paste
… x = 1
…  y = 2
… ^D
```

특정 PureScript 키워드는 새로운 코드 블록을 도입하며, 이 블록 내의 선언은 추가로 들여쓰기 되어야 합니다:

```haskell
example x y z =
  let
    foo = x * y
    bar = y * z
  in
    foo + bar
```

다음 코드는 컴파일되지 않습니다:

```haskell
example x y z =
  let
    foo = x * y
  bar = y * z
  in
    foo + bar
```

더 많은 정보를 원하거나 문제가 발생한 경우, [Syntax](https://github.com/purescript/documentation/blob/master/language/Syntax.md#syntax) 문서를 참조하십시오.

## 타입 정의

PureScript에서 새로운 문제를 해결할 때 좋은 첫 단계는 작업할 값들에 대한 타입 정의를 작성하는 것입니다. 먼저, 주소록의 레코드에 대한 타입을 정의합시다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:Entry}}
```

이는 `Entry`라는 _타입 별칭_ 을 정의합니다.
`Entry` 타입은 이퀄 기호 오른쪽의 타입과 동일합니다: 세 개의 필드를 가진 레코드 타입 – `firstName`, `lastName`, `address`. 두 이름 필드는 `String` 타입을 가지며, `address` 필드는 다음과 같이 정의된 `Address` 타입을 가집니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:Address}}
```

레코드는 다른 레코드를 포함할 수 있습니다.

이제 주소록 데이터 구조에 대한 세 번째 타입 별칭을 정의합시다.
이는 단순히 엔트리의 연결 리스트로 표현됩니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:AddressBook}}
```

`List Entry`는 엔트리의 _배열_ 을 나타내는 `Array Entry`와 다릅니다.

## 타입 생성자와 종(Kinds)

`List`는 _타입 생성자_ 의 예입니다.
값들은 직접 `List` 타입을 가지지 않지만, 대신 `List a` 타입을 가집니다.
즉, `List`는 _타입 인수_ `a`를 받아 새로운 타입 `List a`를 _생성_ 합니다.

함수 적용과 마찬가지로, 타입 생성자는 다른 타입에 단순히 나란히 놓아 적용됩니다: `List Entry` 타입은 실제로 타입 생성자 `List`가 타입 `Entry`에 _적용된_ 것 – 엔트리의 리스트를 나타냅니다.

`List` 타입을 잘못 정의하려고 하면(타입 주석 연산자 `::`를 사용하여), 새로운 종류의 오류를 보게 될 것입니다:

```text
> import Data.List
> Nil :: List
In a type-annotated expression x :: t, the type t must have kind Type
```

이는 _종 오류_ 입니다. 값들이 _타입_ 으로 구분되는 것처럼, 타입들은 _종_ 로 구분되며, 잘못된 종의 타입들은 _종 오류_ 를 발생시킵니다.

`Type`이라는 특별한 종이 있으며, 이는 `Number`와 `String`과 같은 값이 있는 모든 타입의 종을 나타냅니다.

타입 생성자에 대한 종도 있습니다.
예를 들어, `Type -> Type` 종은 타입에서 타입으로의 함수처럼 동작하는 것을 나타내며, 이는 `List`와 같습니다.
그래서 값들은 종이 `Type`인 타입을 가져야 하지만 `List`는 종이 `Type -> Type`이기 때문에 오류가 발생했습니다.

타입의 종을 확인하려면 PSCi에서 `:kind` 명령을 사용하십시오. 예를 들어:

```text
> :kind Number
Type

> import Data.List
> :kind List
Type -> Type

> :kind List String
Type
```

PureScript의 _종 시스템_ 은 다른 흥미로운 종도 지원하며 책의 뒷부분에서 살펴볼 것입니다.

## 정량화된 타입

설명을 위해, 두 개의 인수를 받아 첫 번째 인수를 반환하는 기본 함수를 정의해 봅시다:

```text
> :paste
… constantlyFirst :: forall a b. a -> b -> a
… constantlyFirst = \a b -> a
… ^D
```

> `:type`을 사용하여 `constantlyFirst`의 타입을 묻는 경우 더 자세한 정보를 얻을 수 있습니다:
>
> ```text
> : type constantlyFirst
> forall (a :: Type) (b :: Type). a -> b -> a
> ```
>
> 이 타입 시그니처는  `a`와 `b`가 구체적인 타입이어야 함을 명시적으로 나타냅니다.

`forall` 키워드는 `constantlyFirst`가 _보편적으로 정량화된 타입_ 을 가지고 있음을 나타냅니다.
이는 우리가 `a`와 `b`에 대해 어떤 타입을 대입하든 – `constantlyFirst`가 이 타입들로 동작할 수 있음을 의미합니다.

예를 들어, `a` 타입을 `Int`로, `b` 타입을 `String`으로 선택할 수 있습니다.
이 경우, `constantlyFirst`의 타입을 다음과 같이 _특수화_ 할 수 있습니다:

```text
Int -> String -> Int
```

코드에서 정량화된 타입을 특수화하고자 명시할 필요는 없습니다 – 자동으로 발생합니다.
예를 들어, `constantlyFirst`를 이미 이 타입을 가진 것처럼 사용할 수 있습니다:

```text
> constantlyFirst 3 "ignored"
3
```

`a`와 `b`에 대해 어떤 타입이든 선택할 수 있지만, `constantlyFirst`의 반환 타입은 첫 번째 인수의 타입과 동일해야 합니다 (두 인수 모두 동일한 `a`에 "묶여" 있기 때문입니다):

```text
:type constantlyFirst true "ignored"
Boolean

:type constantlyFirst "keep" 3
String
```

## 주소록 항목 표시

이제 주소록 항목을 문자열로 렌더링하는 첫 번째 함수를 작성해 봅시다.
함수에 타입을 부여하는 것부터 시작할 것입니다.
이는 선택 사항이지만, 문서화의 일환으로 좋은 습관입니다.
실제로, PureScript 컴파일러는 최상위 선언에 타입 주석이 없으면 경고를 제공합니다.
타입 선언은 함수 이름과 그 타입을 `::` 기호로 구분합니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:showEntry_signature}}
```

이 타입 시그니처는 `showEntry`가 `Entry`를 인수로 받아 `String`을 반환하는 함수임을 나타냅니다.
다음은 `showEntry`의 코드입니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:showEntry_implementation}}
```

이 함수는 `Entry` 레코드의 세 필드를 단일 문자열로 연결하며, `address` 필드 안의 레코드를 `String`으로 변환하는 `showAddress` 함수를 사용합니다.
`showAddress`도 유사하게 정의됩니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:showAddress}}
```

함수 정의는 함수의 이름으로 시작하며, 인수 이름 목록이 뒤따릅니다.
함수의 결과는 이퀄 기호 뒤에 지정됩니다.
필드는 점을 사용하여 접근하며, 필드 이름이 뒤따릅니다.
PureScript에서 문자열 연결은 JavaScript와 같은 더하기 연산자가 아닌 다이아몬드 연산자 (`<>`)를 사용합니다.

## 일찍 테스트하고 자주 테스트하기

PSCi 인터랙티브 모드는 즉각적인 피드백을 제공하여 빠른 프로토타이핑을 가능하게 하므로, 이를 사용하여 첫 몇 개의 함수가 예상대로 동작하는지 확인해 봅시다.

먼저 작성한 코드를 빌드하세요:

```text
$ spago build
```

다음으로, PSCi를 로드하고 `import` 명령을 사용하여 새 모듈을 가져옵니다:

```text
$ spago repl

> import Data.AddressBook
```

레코드 리터럴을 사용하여 주소록 항목을 생성할 수 있습니다.
이는 JavaScript에서 익명 객체와 매우 유사합니다:

```text
> address = { street: "123 Fake St.", city: "Faketown", state: "CA" }
```

이제 예제를 적용하여 함수를 적용해 봅시다:

```text
> showAddress address

"123 Fake St., Faketown, CA"
```

`showEntry`를 테스트하기 위해 예제 주소를 포함하는 주소록 항목 레코드를 생성해 봅시다:

```text
> entry = { firstName: "John", lastName: "Smith", address: address }
> showEntry entry

"Smith, John: 123 Fake St., Faketown, CA"
```

## 주소록 생성

이제 주소록을 작업하기 위한 유틸리티 함수를 작성해 봅시다.
빈 주소록을 나타내는 값, 즉 빈 리스트가 필요합니다.

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:emptyBook}}
```

기존 주소록에 값을 삽입하기 위한 함수도 필요합니다.
이 함수를 `insertEntry`라고 부르겠습니다.
타입을 지정하는 것부터 시작해 봅시다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:insertEntry_signature}}
```

이 타입 시그니처는 `insertEntry`가 첫 번째 인수로 `Entry`를, 두 번째 인수로 `AddressBook`을 받아 새로운 `AddressBook`을 반환하는 함수를 나타냅니다.

기존 `AddressBook`을 직접 수정하지 않습니다.
대신, 동일한 데이터를 포함하는 새로운 `AddressBook`을 반환합니다.
따라서 `AddressBook`은 _불변 데이터 구조_ 의 예입니다.
이는 PureScript에서 중요한 개념입니다. - 변경은 코드의 부수 효과로 인해 코드의 동작을 효과적으로 추론하는 것을 방해하기 때문에 가능한 순수 함수와 불변 데이터를 선호합니다.

`insertEntry`를 구현하기 위해 `Data.List`의 `Cons` 함수를 사용할 수 있습니다.
PSCi를 열고 `:type` 명령을 사용하여 그 타입을 확인해 보세요:

```text
$ spago repl

> import Data.List
> :type Cons

forall (a :: Type). a -> List a -> List a
```

이 타입 시그니처는 `Cons`가 어떤 타입 `a`의 값과 동일한 타입의 요소를 가진 리스트를 받은 다음 새로운 리스트를 반환함을 나타냅니다.
이를 `Entry` 타입으로 특수화해 봅시다:

```haskell
Entry -> List Entry -> List Entry
```

그러나 `List Entry`는 `AddressBook`과 동일하므로 이는 다음과 같습니다:

```haskell
Entry -> AddressBook -> AddressBook
```

우리의 경우, 이미 적절한 입력값을 가지고 있습니다: `Entry`와 `AddressBook`.
따라서 `Cons`를 적용하여 새로운 `AddressBook`을 얻을 수 있으며, 이는 우리가 원했던 것입니다!

다음은 `insertEntry`의 구현입니다:

```haskell
insertEntry entry book = Cons entry book
```

이는 이퀄 기호 왼쪽에 두 인수 `entry`와 `book`을 스코프에 가져오고, `Cons` 함수를 적용하여 결과를 생성합니다.

## 커리 함수

PureScript의 함수는 정확히 하나의 인수를 받습니다.
`insertEntry` 함수가 두 개의 인수를 받는 것처럼 보이지만, 이는 _커리 함수_ 의 예입니다.
PureScript에서 모든 함수는 커리 함수로 간주됩니다.

커링은 여러 인수를 받는 함수를 각 인수를 하나씩 받는 함수로 변환하는 것을 의미합니다.
우리가 함수를 호출할 때, 하나의 인수를 전달하고, 또 다른 하나의 인수를 받는 함수를 반환합니다. 모든 인수가 전달될 때까지 계속됩니다.

예를 들어, `5`를 `add` 함수에 전달하면 또 다른 함수를 얻으며, 이 함수는 `int`를 받아 `5`를 더한 다음 결과를 반환합니다:

```haskell
add :: Int -> Int -> Int
add x y = x + y

addFive :: Int -> Int
addFive = add 5
```

`addFive`는 _부분 적용_ 의 결과이며, 이는 여러 인수를 받는 함수에 총 인수의 수보다 적은 인수를 전달하는 것을 의미합니다. 이를 시험해 봅시다:

> `add` 함수를 아직 정의하지 않았다면 다음을 참고하십시오:
>
> ```text
> > import Prelude
> > :paste
>… add :: Int -> Int -> Int
>… add x y = x + y
>… ^D
> ```

```text
> :paste
… addFive :: Int -> Int
… addFive = add 5
… ^D

> addFive 1
6

> add 5 1
6
```

커링과 부분 적용을 더 잘 이해하려면, `add`를 사용하여 다른 함수를 만들어 보십시오.
그리고 `insertEntry`로 돌아갑시다.

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:insertEntry_signature}}
```

타입 시그니처에서 `->` 연산자는 오른쪽으로 결합하므로, 컴파일러는 타입을 다음과 같이 해석합니다:

```haskell
Entry -> (AddressBook -> AddressBook)
```

`insertEntry`는 단일 인수 `Entry`를 받아 새로운 함수를 반환하며, 이 함수는 단일 `AddressBook` 인수를 받아 새로운 `AddressBook`을 반환합니다.

이는 `insertEntry`를 부분적으로 적용하여 첫 번째 인수만 지정할 수 있음을 의미합니다.
예를 들어, PSCi에서 결과 타입을 확인할 수 있습니다:

```text
> :type insertEntry entry

AddressBook -> AddressBook
```

예상대로 반환 타입은 함수였습니다.
반환된 함수를 두 번째 인수에 적용할 수 있습니다:

```text
> :type (insertEntry entry) emptyBook
AddressBook
```

여기서 괄호는 불필요합니다 – 다음은 동일합니다:

```text
> :type insertEntry entry emptyBook
AddressBook
```

이는 함수 적용이 왼쪽으로 결합하므로, 함수 인수를 공백으로 구분하여 하나씩 지정할 수 있음을 설명합니다.

함수 타입의 `->` 연산자는 함수에 대한 _타입 생성자_ 입니다.
이는 함수의 인수 타입과 반환 타입을 가져갑니다 – 각각 왼쪽과 오른쪽 피연산자입니다.

책의 나머지 부분에서는 "두 개의 인수를 가진 함수"와 같은 표현을 사용할 것입니다.
그러나 이는 첫 번째 인수를 받아 두 번째 인수를 받는 함수를 반환하는 커리 함수를 의미함을 이해해야 합니다.

이제 `insertEntry` 정의를 고려해 봅시다:

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry entry book = Cons entry book
```

오른쪽을 명시적으로 괄호로 묶으면 `(Cons entry) book`을 얻습니다.
즉, `insertEntry entry`는 인수를 `Cons entry` 함수로 전달하는 함수입니다.
그러나 두 함수가 모든 입력에 대해 동일한 결과를 가진다면, 이는 동일한 함수입니다!
따라서 양쪽에서 인수 `book`을 제거할 수 있습니다:

```haskell
insertEntry :: Entry -> AddressBook -> AddressBook
insertEntry entry = Cons entry
```

이제 같은 논리로 인수 `entry`를 양쪽에서 제거할 수 있습니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:insertEntry}}
```

이 과정은 _에타 변환_ 이라 불리며, 다른 기술들과 함께 _인수 없는 형태_ 로 함수 재작성에 사용할 수 있습니다.

`insertEntry`의 경우, _에타 변환_ 은 함수 정의를 매우 명확하게 만들었습니다 – "`insertEntry`는 리스트에서 cons 입니다".
그러나 인수 없는 형태가 일반적으로 더 나은지는 논란의 여지가 있습니다.

## 속성 접근자

레코드의 개별 필드(또는 "속성")에 접근하는 함수를 사용하는 일반적인 패턴이 있습니다.
`Entry`에서 `Address`를 추출하는 인라인 함수는 다음과 같이 작성될 수 있습니다:

```haskell
\entry -> entry.address
```

PureScript는 또한 [_속성 접근자_]((https://github.com/purescript/documentation/blob/master/language/Syntax.md#property-accessors)) 줄임 표현을 허용합니다.
여기서 밑줄은 익명 함수 인수로 작동하며, 위의 인라인 함수는 다음과 동일합니다:

```haskell
_.address
```

이는 여러 수준이나 속성에 대해서도 작동하므로, `Entry`와 관련된 도시를 추출하는 함수는 다음과 같이 작성할 수 있습니다:

```haskell
_.address.city
```

예를 들어:

```text
> address = { street: "123 Fake St.", city: "Faketown", state: "CA" }
> entry = { firstName: "John", lastName: "Smith", address: address }
> _.lastName entry
"Smith"

> _.address.city entry
"Faketown"
```

## 주소록 조회

최소한의 주소록 애플리케이션을 위해 구현해야 할 마지막 함수는 이름으로 사람을 조회하고 올바른 `Entry`를 반환하는 함수입니다.
이는 작은 함수들을 구성하여 프로그램을 작성하는 좋은 예가 될 것입니다 – 함수형 프로그래밍의 핵심 아이디어입니다.

주소록을 필터링하여 올바른 이름을 가진 항목만 유지한 다음, 결과 리스트의 첫 번째 요소를 반환할 수 있습니다.

이 접근 방식의 높은 수준의 사양을 사용하여 함수의 타입을 계산할 수 있습니다. 먼저, PSCi를 열고 `filter`와 `head` 함수의 타입을 확인합니다:

```text
$ spago repl

> import Data.List
> :type filter

forall (a :: Type). (a -> Boolean) -> List a -> List a

> :type head

forall (a :: Type). List a -> Maybe a
```

이 두 타입을 이해하기 위해 하나씩 살펴봅시다.

`filter`는 두 개의 인수를 가지는 커리 함수입니다.
첫 번째 인수는 리스트의 요소를 받아 `Boolean` 값을 반환하는 함수입니다.
두 번째 인수는 요소 리스트이며, 반환 값은 또 다른 리스트입니다.

`head`는 리스트를 인수로 받아 이전에 본 적 없는 타입 `Maybe a`를 반환합니다.
`Maybe a`는 타입 `a`의 선택적 값을 나타내며, JavaScript와 같은 언어에서 누락된 값을 나타내기 위해 `null`을 사용하는 것에 대한 타입 안전한 대안을 제공합니다.
이는 이후 챕터에서 더 자세히 살펴볼 것입니다.

`filter`와 `head`의 보편적 타입은 PureScript 컴파일러에 의해 다음 타입으로 _특수화_ 될 수 있습니다:

```haskell
filter :: (Entry -> Boolean) -> AddressBook -> AddressBook

head :: AddressBook -> Maybe Entry
```

조회할 이름과 성을 함수의 인수로 전달해야 할 필요가 있음을 알고 있습니다.

또한 `filter`에 전달할 함수를 정의해야 함을 알고 있습니다.
이 함수를 `filterEntry`라고 부릅시다.
`filterEntry`는 `Entry -> Boolean` 타입을 가집니다.
`filter filterEntry`의 적용은 `AddressBook -> AddressBook` 타입을 가집니다.
이 함수의 결과를 `head` 함수에 전달하면 `Maybe Entry` 타입의 결과를 얻습니다.

이 사실들을 종합하면, `findEntry`라고 부를 함수의 합리적인 타입 시그니처는 다음과 같습니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:findEntry_signature}}
```

이 타입 시그니처는 `findEntry`가 두 개의 문자열, 즉 이름과 성을 받아 `AddressBook`을 받고 선택적 `Entry`를 반환함을 나타냅니다.
선택적 결과는 이름이 주소록에 있는 경우에만 값을 포함합니다.

다음은 `findEntry`의 정의입니다:

```haskell
findEntry firstName lastName book = head (filter filterEntry book)
  where
    filterEntry :: Entry -> Boolean
    filterEntry entry = entry.firstName == firstName && entry.lastName == lastName
```

이 코드를 단계별로 살펴봅시다.

`findEntry`는 `firstName`과 `lastName` 두 개의 문자열 인수와 `book`이라는 `AddressBook`을 스코프에 가져옵니다.

정의의 오른쪽은 `filter`와 `head` 함수가 결합되어 있습니다: 먼저, 엔트리 리스트를 필터링하고 `head` 함수가 결과에 적용됩니다.

필터 함수 `filterEntry`는 `where` 절 내에 보조 선언으로 정의됩니다.
이렇게 하면 `filterEntry` 함수가 함수의 정의 내에서 사용 가능하지만 외부에서는 사용되지 않습니다.
또한, 이는 해당 인수가 포함된 인클로징 함수에 대한 인수에 따라 달라질 수 있습니다.
`filterEntry`는 `firstName`과 `lastName` 인수를 사용하여 지정된 `Entry`를 필터링하기 때문에 여기서 필수적입니다.

최상위 선언과 마찬가지로 `filterEntry`에 대한 타입 시그니처를 지정할 필요는 없었습니다.
그러나 문서화의 한 형태로 권장됩니다.

## 인픽스 함수 적용

지금까지 논의된 대부분의 함수는 _프리픽스_ 함수 적용을 사용했습니다.
함수 이름이 인수 _앞에_ 오는 방식입니다.
예를 들어, `insertEntry` 함수를 사용하여 빈 `AddressBook`에 `Entry` (`john`)를 추가할 때 다음과 같이 쓸 수 있습니다:

```haskell
> book1 = insertEntry john emptyBook
```

그러나 이 챕터에는 인수 _사이에_ 오는 _인픽스_ [이항 연산자](https://github.com/purescript/documentation/blob/master/language/Syntax.md#binary-operators) 의 예도 포함되어 있습니다.
예를 들어, `filterEntry`의 정의에서 `==` 연산자입니다.
이는 인수 _사이에_ 오는 인픽스 연산자입니다.
이러한 인픽스 연산자는 PureScript 소스에서 기본 _프리픽스_ 구현에 대한 인픽스 별칭으로 정의됩니다.
예를 들어, `==`는 프리픽스 `eq` 함수의 인픽스 별칭으로 다음과 같이 정의됩니다:

```haskell
infix 4 eq as ==
```

따라서 `filterEntry`에서 `entry.firstName == firstName`은 `eq entry.firstName firstName`으로 대체될 수 있습니다.
이 섹션의 뒷부분에서 인픽스 연산자를 정의하는 몇 가지 예를 더 다룰 것입니다.

프리픽스 함수를 인픽스 위치에서 연산자로 사용하는 것이 더 읽기 쉬운 코드로 이어지는 상황이 있습니다.
한 가지 예는 `mod` 함수입니다:

```text
> mod 8 3
2
```

위의 사용법은 괜찮지만 읽기에는 어색합니다.
더 익숙한 표현은 "eight mod three"이며, 이는 프리픽스 함수를 역따옴표(\`)로 묶어서 얻을 수 있습니다:

```text
> 8 `mod` 3
2
```

같은 방식으로 `insertEntry`를 역따옴표로 묶으면 인픽스 연산자가 되어 다음과 같이 `book1`과 `book2`는 동일해집니다:

```haskell
book1 = insertEntry john emptyBook
book2 = john `insertEntry` emptyBook
```

여러 개의 `insertEntry`를 프리픽스 함수(`book3`) 또는 인픽스 연산자(`book4`)로 사용하여 여러 항목이 있는 `AddressBook`을 만들 수 있습니다:

```haskell
book3 = insertEntry john (insertEntry peggy (insertEntry ned emptyBook))
book4 = john `insertEntry` (peggy `insertEntry` (ned `insertEntry` emptyBook))
```

`insertEntry`에 대한 인픽스 연산자 별칭(또는 동의어)을 정의할 수도 있습니다.
임의로 `++`를 이 연산자로 선택하고, [우선순위](https://github.com/purescript/documentation/blob/master/language/Syntax.md#precedence) 를 `5`로 지정하고, 오른쪽 [결합](https://github.com/purescript/documentation/blob/master/language/Syntax.md#associativity) 을 `infixr`를 사용하여 만듭니다:

```haskell
infixr 5 insertEntry as ++
```

이 새로운 연산자를 사용하면 위의 `book4` 예제를 다음과 같이 다시 쓸 수 있습니다:

```haskell
book5 = john ++ (peggy ++ (ned ++ emptyBook))
```

새로운 `++` 연산자의 오른쪽 결합성은 의미를 변경하지 않고 괄호를 제거할 수 있게 합니다:

```haskell
book6 = john ++ peggy ++ ned ++ emptyBook
```

괄호를 제거하는 또 다른 방법은 `apply`의 인픽스 연산자 `$`를 사용하는 것입니다.
표준 프리픽스 함수와 함께 사용됩니다.

예를 들어, 이전의 `book3` 예제를 다음과 같이 다시 쓸 수 있습니다:

```haskell
book7 = insertEntry john $ insertEntry peggy $ insertEntry ned emptyBook
```

괄호를 `$`로 대체하는 것은 일반적으로 타이핑하기 더 쉽고 (논란의 여지가 있지만) 읽기 더 쉽습니다.
이 기호의 의미를 기억하는 방법으로는 달러 기호를 두 괄호로 그려서 교차된 것으로 생각할 수 있습니다.
이 두 괄호는 줄이 그어져 있으며 이제 괄호가 필요하지 않음을 나타냅니다.

참고로 `$`는 언어에 하드코딩된 특별한 문법이 아닙니다.
이는 `Data.Function`에 다음과 같이 정의된 일반 함수 `apply`의 인픽스 연산자입니다:

```haskell
apply :: forall a b. (a -> b) -> a -> b
apply f x = f x

infixr 0 apply as $
```

`apply` 함수는 함수(타입 `(a -> b)`)를 첫 번째 인수로 받고 값(타입 `a`)을 두 번째 인수로 받아 그 값으로 함수를 호출합니다.
이 함수가 의미 있는 기여를 하지 않는 것처럼 보인다면, 완전히 맞습니다!
프로그램은 논리적으로 동일합니다(참고: [참조 투명성](https://en.wikipedia.org/wiki/Referential_transparency)).
이 함수의 구문 유틸리티는 인픽스 연산자에 할당된 특수 속성에서 비롯됩니다.
`$`는 오른쪽 결합(`infixr`), 낮은 우선순위(`0`) 연산자로, 깊게 중첩된 적용을 위한 괄호 집합을 제거할 수 있게 합니다.

다른 괄호 제거 기회는 이전의 `findEntry` 함수에서 `$` 연산자를 사용하는 것입니다:

```haskell
findEntry firstName lastName book = head $ filter filterEntry book
```

다음 섹션에서 "함수 합성"으로 이 줄을 더 우아하게 다시 작성하는 방법을 볼 것입니다.

간결한 인픽스 연산자 별칭을 프리픽스 함수로 사용하려면 괄호로 묶을 수 있습니다:

```text
> 8 + 3
11

> (+) 8 3
11
```

또한, 연산자를 부분적으로 적용하려면 표현식을 괄호로 묶고 `_`를 피연산자로 사용하여 [연산자 섹션](https://github.com/purescript/documentation/blob/master/language/Syntax.md#operator-sections)으로 만들 수 있습니다.
이는 간단한 익명 함수를 만드는 더 편리한 방법으로 생각할 수 있습니다 (다음 예제에서는, 그 익명 함수를 이름에 바인딩하므로 더 이상 익명은 아니지만):

```text
> add3 = (3 + _)
> add3 2
5
```

요약하자면, 다음은 `5`를 인수로 더하는 함수의 동일한 정의입니다:

```haskell
add5 x = 5 + x
add5 x = add 5 x
add5 x = (+) 5 x
add5 x = 5 `add` x
add5   = add 5
add5   = \x -> 5 + x
add5   = (5 + _)
add5 x = 5 `(+)` x  -- Yo Dawg, I herd you like infix, so we put infix in your infix!
```

## 함수 합성

`insertEntry` 함수를 에타 변환을 사용하여 단순화할 수 있었던 것처럼, `findEntry`의 정의를 인수에 대해 추론하여 단순화할 수 있습니다.

`book` 인수는 `filter filterEntry` 함수에 전달되고, 이 함수의 결과는 `head` 함수에 전달됩니다.
즉, `book`은 `filter filterEntry`와 `head` 함수의 _합성_ 에 전달됩니다.

PureScript에서 함수 합성 연산자는 `<<<`와 `>>>`입니다.
첫 번째는 "역방향 합성", 두 번째는 "순방향 합성"입니다.

우리는 `findEntry`의 오른쪽을 이 연산자 중 하나를 사용하여 다시 작성할 수 있습니다.
역방향 합성을 사용하면 오른쪽은 다음과 같습니다:

```haskell
(head <<< filter filterEntry) book
```

이 형태에서는 앞서 언급한 에타 변환 기법을 적용하여 `findEntry`의 최종 형태에 도달할 수 있습니다:

```haskell
{{#include ../exercises/chapter3/src/Data/AddressBook.purs:findEntry_implementation}}
    ...
```

동일하게 유효한 오른쪽은 다음과 같습니다:

```haskell
filter filterEntry >>> head
```

어느 쪽이든, 이는 `findEntry` 함수의 명확한 정의를 제공합니다: "`findEntry`는 필터링 함수와 `head` 함수의 합성입니다".

어떤 정의가 이해하기 더 쉬운지는 여러분에게 달려 있지만, 함수들을 이런 방식으로 구성 요소로 생각하는 것이 종종 유용합니다: 각 함수는 단일 작업을 실행하고, 함수 합성을 사용하여 솔루션이 조립됩니다.

## 연습 문제

1. (쉬움) `findEntry` 함수의 주요 서브 표현식 각각의 타입을 적어보면서 `findEntry` 함수에 대한 이해를 테스트하세요. 예를 들어, `head` 함수의 사용된 타입은 `AddressBook -> Maybe Entry`로 특수화됩니다. _참고_: 이 연습 문제에는 테스트가 없습니다.
2. (중간) 거리 주소를 주어진 `Entry`를 조회하는 `findEntryByStreet :: String -> AddressBook -> Maybe Entry` 함수를 작성하세요. _힌트_ `findEntry`의 기존 코드를 재사용합니다. PSCi에서 함수 테스트를 실행하고 `spago test`를 실행하여 테스트하세요.
3. (중간) `findEntryByStreet`를 다시 작성하여 `filterEntry`를 `<<<` 또는 `>>>`를 사용하는 구성으로 대체하세요: 속성 접근자 (`_.` 표기법 사용)와 주어진 문자열 인수가 주어진 거리 주소와 같은지 테스트하는 함수의 구성으로 대체하세요.
4. (중간) `AddressBook`에서 이름이 존재하는지 테스트하고 불리언 값을 반환하는 `isInBook` 함수를 작성하세요. _힌트_: 리스트가 비어 있는지 여부를 테스트하는 `Data.List.null` 함수의 타입을 PSCi에서 찾아보세요.
5. (어려움) "중복" 주소록 항목을 제거하는 `removeDuplicates` 함수를 작성하세요. 항목이 동일한 이름과 성을 공유하는 경우 주소 필드를 무시하고 중복으로 간주합니다. _힌트_: 리스트의 중복 요소를 제거하는 `Data.List.nubByEq` 함수의 타입을 PSCi에서 찾아보세요. 중복 세트의 첫 번째 요소 (리스트 헤드에 가장 가까운)가 유지됩니다.

## 결론

이번 챕터에서는 몇 가지 새로운 함수형 프로그래밍 개념을 다루었고, 다음을 배우는 방법을 익혔습니다:

- 인터랙티브 모드 PSCi를 사용하여 함수를 실험하고 아이디어를 테스트합니다.
- 타입을 정확성 도구와 구현 도구로 사용합니다.
- 커리 함수를 사용하여 여러 인수의 함수를 나타냅니다.
- 더 작은 구성 요소를 조합하여 프로그램을 작성합니다.
- `where` 표현식을 사용하여 코드를 깔끔하게 구조화합니다.
- `Maybe` 타입을 사용하여 null 값을 피합니다.
- 에타 변환 및 함수 합성과 같은 기술을 사용하여 코드를 명확한 사양으로 리팩토링합니다.

다음 챕터에서는 이 아이디어를 바탕으로 더욱 발전할 것입니다.
