# 소개

## 함수형 Javascript

함수형 프로그래밍 기법은 이미 오랫동안 자바스크립트에 등장해 왔습니다:

- [UnderscoreJS](https://underscorejs.org)와 같은 라이브러리는 `map`, `filter`, `reduce`와 같은 검증된 함수를 활용하여 작은 프로그램들을 조합해서 더 큰 프로그램을 만들 수 있게 합니다:

    ```javascript
    var sumOfPrimes =
        _.chain(_.range(1000))
         .filter(isPrime)
         .reduce(function(x, y) {
             return x + y;
         })
         .value();
    ```

- NodeJS에서의 비동기 프로그래밍은 콜백을 정의하기 위해 함수가 일급 값으로 사용되는 것에 크게 의존합니다.

    ```javascript
    import { readFile, writeFile } from 'fs'

    readFile(sourceFile, function (error, data) {
      if (!error) {
        writeFile(destFile, data, function (error) {
          if (!error) {
            console.log("File copied");
          }
        });
      }
    });
    ```

- [React](https://reactjs.org) 및 [virtual-dom](https://github.com/Matt-Esch/virtual-dom)과 같은 라이브러리는 애플리케이션 상태에 대한 순수 함수로 뷰를 모델링합니다.

함수는 단순한 형태의 추상화를 가능하게 하여 큰 생산성 향상을 가져올 수 있습니다.
그러나 Javascript에서의 함수형 프로그래밍에는 단점이 있습니다: Javascript는 장황하고, 타입이 없으며, 강력한 형태의 추상화가 부족합니다.
또한 제한되지 않은 Javascript 코드는 등식 추론을 매우 어렵게 만듭니다.

PureScript는 이러한 문제를 해결하려는 프로그래밍 언어입니다.
가벼운 문법을 특징으로 하여 표현력이 뛰어난 코드를 작성할 수 있으며, 여전히 명확하고 읽기 쉽습니다.
또한 풍부한 타입 시스템을 사용하여 강력한 추상화를 지원합니다.
JavaScript 또는 JavaScript로 컴파일되는 다른 언어와 상호 운용할 때 중요한 빠르고 이해하기 쉬운 코드를 생성합니다.
전체적으로, PureScript는 순수 함수형 프로그래밍의 이론적 강점과 Javascript의 빠르고 느슨한 프로그래밍 스타일 사이에서 매우 실용적인 균형을 이루고 있음을 이해하셨으면 합니다.

> PureScript는 JavaScript뿐만 아니라 다른 백엔드도 타겟팅할 수 있지만, 이 책에서는 웹 브라우저와 노드 환경을 타겟팅하는 데 중점을 두고 있습니다.

## 타입과 타입 추론

정적 타입 언어와 동적 타입 언어에 대한 논쟁은 잘 문서화되어 있습니다.
PureScript는 정적 타입 언어로, 올바른 프로그램은 컴파일러에 의해 타입을 부여받을 수 있으며, 이는 프로그램의 동작을 나타냅니다.
반면, 타입을 부여받을 수 없는 프로그램은 잘못된 프로그램으로 간주되어 컴파일러에 의해 거부됩니다.
PureScript에서는 동적 타입 언어와 달리 타입이 오직 컴파일 시점에만 존재하며 런타임에는 표현되지 않습니다.

중요한 점은, 여러 면에서 PureScript의 타입은 Java나 C#과 같은 다른 언어에서 볼 수 있는 타입과 다르다는 것입니다.
높은 수준에서는 같은 목적을 수행하지만, PureScript의 타입은 ML과 Haskell과 같은 언어에서 영감을 받았습니다.
PureScript의 타입은 표현력이 뛰어나 개발자가 프로그램에 대해 강력한 주장을 표현할 수 있습니다.
가장 중요한 것은, PureScript의 타입 시스템은 타입 추론을 지원한다는 점입니다.
타입 추론이 있으면 다른 언어보다 훨씬 적은 명시적 타입 주석을 요구하므로 타입 시스템을 성가신 존재가 아닌 도구로 만듭니다.
간단한 예로, 다음 코드는 `Number` 타입에 대한 언급 없이 숫자를 정의합니다:

```haskell
iAmANumber =
  let square x = x * x
  in square 42.0
```

좀 더 복잡한 예는 컴파일러가 알 수 없는 타입이 존재하더라도 타입의 정확성을 타입 주석 없이 확인할 수 있음을 보여줍니다:

```haskell
iterate f 0 x = x
iterate f n x = iterate f (n - 1) (f x)
```

여기서 `x`의 타입은 알려지지 않았지만, 컴파일러는 `iterate`가 타입 시스템의 규칙을 준수한다는 것을 확인할 수 있습니다.

이 책에서 저는 정적 타입이 단순히 프로그램의 올바름에 대한 확신을 주는 수단일 뿐만 아니라 자체적으로 개발에 도움이 된다는 것을 여러분에게 설득하거나 재확인시키려고 합니다.
Javascript에서 큰 코드 베이스를 리팩토링하는 것은 가장 단순한 추상화를 사용할 때조차도 어렵지만, 표현력 있는 타입 시스템과 타입 검사기는 리팩토링을 즐겁고 상호작용적인 경험으로 만들 수 있습니다.

또한, 타입 시스템이 제공하는 안전망은 더 고급 형태의 추상화를 가능하게 합니다.
사실, PureScript는 타입에 의해 구동되는 강력한 형태의 추상화를 제공하는데, 이는 Haskell에서 인기 있는 타입 클래스입니다.

## 다중 언어 웹 프로그래밍

함수형 프로그래밍은 데이터 분석, 파싱, 컴파일러 구현, 제네릭 프로그래밍, 병렬성 등 특히 성공적인 사례가 많습니다.

PureScript와 같은 함수형 언어로 엔드 투 엔드 애플리케이션 개발을 할 수 있습니다.
PureScript는 값과 함수에 대한 타입을 제공하여 기존 Javascript 코드를 가져와 정규 PureScript 코드에서 사용할 수 있게 합니다.
우리는 이 접근 방식을 책 후반부에서 살펴볼 것입니다.

그러나 PureScript의 강점 중 하나는 Javascript를 타겟팅하는 다른 언어와의 상호 운용성입니다.
다른 접근 방식으로는 애플리케이션 개발의 하위 집합에 PureScript를 사용하고 나머지 자바스크립트를 작성하는 방법이 있습니다.

여기 몇 가지 예시가 있습니다:

- 핵심 로직을 PureScript로 작성하고, 사용자 인터페이스를 Javascript로 작성합니다.
- 애플리케이션을 Javascript 또는 다른 JS 컴파일 언어로 작성하고, 테스트를 PureScript로 작성합니다.
- 기존 애플리케이션의 사용자 인터페이스 테스트를 자동화하기 위해 PureScript를 사용합니다.

이 책에서는 PureScript로 작은 문제를 해결하는 데 중점을 둡니다.
솔루션은 더 큰 애플리케이션에 통합될 수 있지만, PureScript 코드를 Javascript에서 호출하는 방법과 그 반대의 경우도 살펴볼 것입니다.

## 전제 조건

이 책의 소프트웨어 요구 사항은 최소한입니다:
첫 번째 장에서는 처음부터 개발 환경을 설정하는 과정을 안내하며, 사용할 도구는 대부분의 최신 운영 체제의 표준 저장소에서 사용할 수 있습니다.

PureScript 컴파일러 자체는 바이너리 배포판으로 다운로드하거나 최신 GHC Haskell 컴파일러 설치가 실행 중인 시스템에서 소스에서 빌드할 수 있으며, 다음 장에서 이 과정을 설명할 것입니다.

이 책 버전의 코드는 PureScript 컴파일러 `0.15.*` 버전과 호환됩니다.

## 독자에 대하여

Javascript의 기본에 익숙하다고 가정합니다.
자바스크립트 생태계의 공통 도구(NPM 및 Gulp 등)에 대한 사전 지식은 표준 설정을 자신의 필요에 맞게 커스터마이즈하려는 경우 유용할 수 있지만, 그러한 지식은 필수는 아닙니다.

함수형 프로그래밍에 대한 사전 지식은 필요하지 않지만, 있다면 도움이 될 것입니다.
새로운 아이디어는 실용적인 예제와 함께 제공되므로 함수형 프로그래밍에서 사용할 개념에 대한 직관을 형성할 수 있을 것입니다.

Haskell 프로그래밍 언어에 익숙한 독자는 이 책에서 소개되는 많은 아이디어와 구문을 인식할 것입니다.
PureScript는 Haskell에 많은 영향을 받았기 때문입니다.
그러나 이러한 독자는 PureScript와 Haskell 사이에 중요한 차이점이 여러 가지 있다는 점을 이해해야 합니다.
항상 한 언어에서 다른 언어로 아이디어를 적용하는 것이 적절하지 않을 수 있지만, 이 책에서 제시된 많은 개념은 Haskell에서 어떤 해석을 가질 것입니다.

## 이 책을 읽는 방법

이 책의 각 장은 대체로 독립적입니다.
함수형 프로그래밍 경험이 거의 없는 초보자는 장을 순서대로 진행하는 것이 좋습니다.
처음 몇 장은 책 후반의 자료를 이해하는 데 필요한 기초를 다집니다.
함수형 프로그래밍 개념에 익숙한 독자(특히 ML이나 Haskell과 같은 강력한 타입 언어에 경험이 있는 독자)는 책의 앞부분을 읽지 않고도 후반 장의 코드를 전반적으로 이해할 수 있을 것입니다.

각 장은 단일 실용 예제에 중점을 두어 새로운 아이디어를 도입하기 위한 동기를 제공합니다.
각 장의 코드는 책의 [GitHub 저장소](https://github.com/purescript-contrib/purescript-book)에서 확인할 수 있습니다.
일부 장에는 장의 소스 코드에서 발췌한 코드 스니펫이 포함될 수 있지만, 전체적인 이해를 위해서는 책의 자료와 함께 저장소의 소스 코드를 읽어야 합니다.
더 긴 섹션은 자신의 이해를 테스트하기 위해 인터랙티브 모드 PSCi에서 실행할 수 있는 더 짧은 스니펫을 포함할 것입니다.

코드 샘플은 다음과 같이 모노스페이스 글꼴로 표시됩니다:

```haskell
module Example where

import Effect.Console (log)

main = log "Hello, World!"
```

명령줄에서 입력해야 하는 명령은 달러 기호로 시작합니다:

```text
$ spago build
```

대개 이러한 명령은 Linux/Mac OS 사용자에게 맞춰져 있으므로, Windows 사용자는 파일 구분 기호를 수정하거나 셸 빌트인을 Windows의 동등한 것으로 교체하는 등 약간의 변경이 필요할 수 있습니다.

PSCi 인터랙티브 모드 프롬프트에서 입력해야 하는 명령은 각도 기호로 시작합니다:

```text
> 1 + 2
3
```

각 장에는 난이도 수준이 표시된 연습 문제가 포함될 것입니다.
자료를 완전히 이해하려면 각 장의 연습 문제를 시도하는 것이 좋습니다.

이 책은 초보자를 위한 PureScript 언어 입문서를 제공하는 것을 목표로 하지만, 문제에 대한 템플릿 솔루션 목록을 제공하는 책은 아닙니다.
초보자에게 이 책은 재미있는 도전이 될 것이며, 자료를 읽고 연습 문제를 시도하고 가장 중요한 것은 자신의 코드를 작성해 보려는 노력이 가장 큰 도움이 될 것입니다.

## 도움 받기

어느 시점에서 막히게 되면, PureScript 학습을 위한 온라인 리소스가 많이 있습니다:

- [PureScript Discord 서버](https://discord.gg/vKn9up84bp)는 문제에 대한 대화를 나누기에 좋은 장소입니다. 이 서버는 PureScript에 관한 대화에 전념하고 있습니다.
- [PureScript Discourse 포럼](https://discourse.purescript.org/)도 일반적인 문제에 대한 해결책을 검색하기에 좋은 장소입니다.
- [PureScript: Jordan's Reference](https://github.com/jordanmartinez/purescript-jordans-reference)는 매우 깊이 있게 다루는 대안 학습 리소스입니다. 이 책의 개념을 이해하기 어렵다면 해당 참조의 해당 섹션을 읽어 보세요.
- [Pursuit](https://pursuit.purescript.org)는 PureScript 타입과 함수를 검색할 수 있는 데이터베이스입니다. Pursuit의 도움말 페이지를 읽고 [어떤 종류의 검색을 할 수 있는지 알아보세요](https://pursuit.purescript.org/help/users).
- 비공식 [PureScript Cookbook](https://github.com/JordanMartinez/purescript-cookbook)는 "X를 어떻게 하나요?" 유형의 질문에 대해 코드로 답변을 제공합니다.
- [PureScript 문서 저장소](https://github.com/purescript/documentation)는 PureScript 개발자와 사용자가 작성한 다양한 주제에 대한 기사와 예제를 수집합니다.
- [PureScript 웹사이트](https://www.purescript.org)에는 코드 샘플, 비디오 및 초보자를 위한 기타 리소스를 포함한 여러 학습 리소스에 대한 링크가 포함되어 있습니다.
- [Try PureScript!](https://try.purescript.org)는 PureScript 코드를 웹 브라우저에서 컴파일할 수 있게 해주며, 간단한 코드 예제가 포함되어 있습니다.

예제를 읽으면서 학습하는 것을 선호하는 경우, [purescript](https://github.com/purescript), [purescript-node](https://github.com/purescript-node), 및 [purescript-contrib](https://github.com/purescript-contrib) GitHub 조직에는 많은 PureScript 코드 예제가 있습니다.

## 저자 소개

저는 PureScript 컴파일러의 최초 개발자입니다.
저는 캘리포니아 로스앤젤레스에 거주하며, 8비트 개인용 컴퓨터인 Amstrad CPC에서 BASIC으로 어린 시절 프로그래밍을 시작했습니다.
그 이후로 Java, Scala, C#, F#, Haskell, PureScript 등 다양한 프로그래밍 언어로 전문적으로 일해왔습니다.

직업 경력 초기에 함수형 프로그래밍과 수학과의 연결성을 이해하게 되었고, Haskell 프로그래밍 언어를 사용하여 함수형 개념을 배우는 것을 즐겼습니다.

PureScript 컴파일러 작업을 시작한 것은 Javascript와의 경험에서 비롯되었습니다.
Haskell과 같은 언어에서 익힌 함수형 프로그래밍 기법을 사용하면서도 이를 적용할 원칙 있는 환경이 필요하다는 것을 느꼈습니다.
당시의 해결책으로는 Haskell의 의미를 보존하면서 Javascript로 컴파일하려는 다양한 시도(Fay, Haste, GHCJS)가 있었지만, 저는 문제를 다른 관점에서 접근하여 Javascript의 의미를 유지하면서 Haskell과 같은 언어의 문법과 타입 시스템을 즐기는 것이 얼마나 성공적일 수 있는지 보고 싶었습니다.

저는 [블로그](https://blog.functorial.com)를 운영하며, [트위터](https://twitter.com/paf31)에서 저를 찾을 수 있습니다.

## 감사의 말

PureScript가 현재 상태에 도달하는 데 도움을 준 많은 기여자들에게 감사드립니다.
컴파일러, 도구, 라이브러리, 문서 및 테스트에 대한 엄청난 공동의 노력이 없었다면 프로젝트는 확실히 실패했을 것입니다.

이 책의 표지에 등장하는 PureScript 로고는 Gareth Hughes가 만든 것으로, [Creative Commons Attribution 4.0 라이선스](https://creativecommons.org/licenses/by/4.0/)의 조건에 따라 이곳에 감사히 재사용되었습니다.

마지막으로, 이 책의 내용에 대해 피드백과 교정을 해주신 모든 분들께 감사드립니다.
