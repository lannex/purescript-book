# 시작하기

## 챕터 목표

이 챕터에서는 PureScript 개발 환경을 설정하고, 몇 가지 연습 문제를 풀고, 책에서 제공하는 테스트를 사용하여 답을 확인할 것입니다.
[이 챕터의 비디오 워크스루](https://www.youtube.com/watch?v=GPjPwb6d-70)가 학습 스타일에 더 맞는다면 참고하시기 바랍니다.

## 환경 설정

먼저, 환경을 설정하고 언어에 대해 몇 가지 기본적인 내용을 배우기 위해 [Getting Started Guide](https://github.com/purescript/documentation/blob/master/guides/Getting-Started.md)를 따라 진행하세요.
[Project Euler](http://projecteuler.net/problem=1) 문제의 예제 해결 코드가 혼란스럽거나 낯선 구문을 포함하고 있더라도 걱정하지 마세요.
다음 챕터에서 이를 모두 자세히 다룰 것입니다.

### 에디터 지원

PureScript를 작성하기 위해 선호하는 에디터를 사용할 수 있습니다 (예를 들어, 책의 연습 문제를 해결하기 위해).
[에디터 지원 문서](https://github.com/purescript/documentation/blob/master/ecosystem/Editor-and-tool-support.md#editor-support)를 참조하세요.

> 일부 에디터는 완전한 IDE 지원을 위해 열린 프로젝트의 루트에 `spago.dhall` 파일이 있어야 합니다.
> 예를 들어, 이 챕터의 연습 문제를 작업하려면 `chapter2` 디렉토리를 여세요.
>
> VS Code를 사용하는 경우, 제공된 작업 공간을 사용하여 모든 챕터를 동시에 열 수 있습니다.

## 연습 문제 풀기

필요한 개발 도구를 설치했으니 이 책의 레포를 클론하세요.

```sh
git clone https://github.com/purescript-contrib/purescript-book.git
```

책 레포에는 각 챕터에 동반되는 PureScript 예제 코드와 연습 문제의 단위 테스트가 포함되어 있습니다.
연습 문제 해결을 준비하기 위해 약간의 초기 설정이 필요합니다.
`prepareExercises.sh` 스크립트를 사용하여 이 과정을 간소화하세요:

```sh
cd purescript-book
./scripts/prepareExercises.sh
git add .
git commit --all --message "Exercises ready to be solved"
```

이제 이 챕터의 테스트를 실행하세요:

```sh
cd exercises/chapter2
spago test
```

다음과 같은 성공적인 테스트 출력이 표시되어야 합니다:

```sh
→ Suite: Euler - Sum of Multiples
  ✓ Passed: below 10
  ✓ Passed: below 1000

All 2 tests passed! 🎉
```

`answer` 함수(`src/Euler.purs`에 있음)는 임의의 정수 이하의 3과 5의 배수를 찾도록 수정되었습니다.
이 함수의 테스트 스위트(`test/Main.purs`에 위치)는 초기 시작 가이드의 테스트보다 더 포괄적입니다.
이 초기 챕터를 읽는 동안 이 테스트 프레임워크 코드가 어떻게 작동하는지 이해하지 못해도 괜찮습니다.

책의 나머지 부분에는 많은 연습 문제가 포함되어 있습니다.
`Test.MySolutions` 모듈(`test/MySolutions.purs`)에 솔루션을 작성하면 제공된 테스트 스위트로 작업을 확인할 수 있습니다.

다음 연습 문제를 테스트 주도 개발 스타일로 함께 해결해 봅시다.

## 연습 문제

1. (중간) 직각 삼각형의 두 다른 변의 길이를 주어질 때 대각선(또는 빗변)의 길이를 계산하는 `diagonal` 함수를 작성하세요.

## 해결 방법

이 연습 문제의 테스트를 활성화하는 것으로 시작하겠습니다.
블록 주석의 시작 부분을 몇 줄 아래로 이동하세요. 블록 주석은 `{-`로 시작하고 `-}`로 끝납니다:

```hs
{{#include ../exercises/chapter2/test/Main.purs:diagonalTests}}
    {-  Move this block comment starting point to enable more tests
```

이제 테스트를 실행하려고 하면 `diagonal` 함수를 아직 구현하지 않았기 때문에 컴파일 오류가 발생합니다.

```sh
$ spago test

Error found:
in module Test.Main
at test/Main.purs:21:27 - 21:35 (line 21, column 27 - line 21, column 35)

  Unknown value diagonal
```

이 함수의 잘못된 버전이 어떻게 동작하는지 먼저 살펴봅시다.
`test/MySolutions.purs`에 다음 코드를 추가하세요:

```hs
import Data.Number (sqrt)

diagonal w h = sqrt (w * w + h)
```

그리고 `spago test`를 실행하여 작업을 확인하세요:

```hs
→ Suite: diagonal
  ☠ Failed: 3 4 5 because expected 5.0, got 3.605551275463989
  ☠ Failed: 5 12 13 because expected 13.0, got 6.082762530298219

2 tests failed:
```

어이쿠, 이것은 올바르지 않습니다. 피타고라스 공식의 올바른 적용으로 이를 수정해 봅시다:

```hs
{{#include ../exercises/chapter2/test/no-peeking/Solutions.purs:diagonal}}
```

다시 `spago test`를 실행하면 모든 테스트가 통과된 것을 볼 수 있습니다:

```sh
→ Suite: Euler - Sum of Multiples
  ✓ Passed: below 10
  ✓ Passed: below 1000
→ Suite: diagonal
  ✓ Passed: 3 4 5
  ✓ Passed: 5 12 13

All 4 tests passed! 🎉
```

성공입니다! 이제 혼자서 다음 연습 문제를 시도할 준비가 되었습니다.

## 연습 문제

1. (쉬움) 주어진 반지름으로 원의 면적을 계산하는 `circleArea` 함수를 작성하세요.
   `Numbers` 모듈에 정의된 `pi` 상수를 사용하세요. _힌트_: `Data.Number` 문을 수정하여 `pi`를 임포트하는 것을 잊지 마세요.
2. (중간) `Int`를 받아서 `100`으로 나눈 나머지를 반환하는 `leftoverCents` 함수를 작성하세요.
   `rem` 함수를 사용하세요. 이 함수의 사용법과 어떤 모듈에서 임포트해야 하는지 배우기 위해 [Pursuit](https://pursuit.purescript.org/)를 검색하세요.
   _참고_: IDE에서 자동 완성 제안을 수락하면 이 함수를 자동으로 임포트할 수 있습니다.

## 결론

이 챕터에서는 PureScript 컴파일러와 Spago 도구를 설치했습니다.
또한, 연습 문제에 대한 해결책을 작성하고 이를 올바르게 확인하는 방법을 배웠습니다.

앞으로의 챕터에는 더 많은 연습 문제가 있을 것이며, 이를 해결하는 과정에서 자료를 학습할 수 있습니다.
연습 문제 중 하나라도 해결에 어려움을 겪으면, [도움 받기](https://book.purescript.org/chapter1.html#getting-help) 섹션에 나열된 커뮤니티 리소스에 문의하거나 이 [책의 레포](https://github.com/purescript-contrib/purescript-book/issues)에 이슈를 제기하세요.
연습 문제를 더 접근하기 쉽게 만들기 위한 독자 피드백은 책 개선에 도움이 됩니다.

챕터의 모든 연습 문제를 해결하면 `no-peeking/Solutions.purs`에 있는 답과 비교할 수 있습니다.
정직하게 문제를 해결하려고 노력하지 않고 답을 보지 마세요.
막히더라도 먼저 커뮤니티 회원에게 도움을 요청해 보세요.
우리는 여러분에게 문제를 스포일러하지 않고 작은 힌트를 주고 싶습니다.
더 우아한 해결책(다룬 내용만을 요구하는)이 있다면 PR을 보내 주세요.

레포는 지속적으로 수정되고 있으니, 각 새로운 챕터를 시작하기 전에 업데이트를 확인하세요.
