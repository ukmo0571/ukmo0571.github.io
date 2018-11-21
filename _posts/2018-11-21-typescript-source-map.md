---
title: "sourceMap으로 Error 추적하기"
date: 2018-11-21 23:51:00 +0900
categories: typescript ts error stack
---

## 시작
Front-end, Back-end를 막론하고 TypeScript없이 개발하는 것은 상상할 수 없을 정도로 빠져있다. JavaScript에 비해 편리한 점이 많지만, 한편으로는 신경 써줘야 할 부분도 한두 가지가 아닌 것 같다. 그 중 한 가지는 Error 추적이다.

```ts
class Test {
  constructor() {
    throw new Error('Error!')
  }
}

const test = new Test()
```

위 코드를 JavaScript에서 실행했을 경우 아래와 같이 에러가 발생한다.

```bash
Error: Error!
    at new Test (/Users/ukmo/Desktop/ts-error/index.js:3:11)
    at Object.<anonymous> (/Users/ukmo/Desktop/ts-error/index.js:7:14)
    ...
```

stack을 읽어보면 `Error!`는 `index.js` 파일의 3번째 라인에서 발생한 것임을 확인할 수 있다. 이 코드 그대로 TypeScript로 작성하고 `tsc`로 변환 후 실행했다.

```bash
Error: Error!
    at new Test (/Users/ukmo/Desktop/ts-error/dist/index.js:4:15)
    at Object.<anonymous> (/Users/ukmo/Desktop/ts-error/dist/index.js:8:12)
    ...
```

`dist/index.js` 파일의 4번째 줄에서 에러가 발생했다. 변환된 JavaScript 파일의 내용은 아래와 같다.

```ts
"use strict";
var Test = /** @class */ (function () {
    function Test() {
        throw new Error('Error!');
    }
    return Test;
}());
var test = new Test();
```

예제 코드가 간단하기 때문에 바로 알 수 있지만, 상용 프로젝트에선 훨씬 더 복잡하기 때문에 변환된 JavaScript파일을 보고 에러를 추적할 수는 없다.

어떻게 하면 stack만 보고 바로 추적할 수 있지 고민해봤는데 해결 방법은 생각보다 훨씬 간단했다.

`sourceMap`은 이 고민을 해결해준다. 방법은 아래와 같다.

## 적용

1. *tsconfig.json* 파일에서 `sourceMap`을 `true`로 설정한다. 
    ```json
    {
      "compilerOptions": {
        "sourceMap": true
      }
    }
    ```
2. `tsc`로 변환하면 `*.map` 파일이 생성된다. 이 파일이 브릿지 역할을 한다.
3. `source-map-support` 모듈을 설치하고 import/require 코드로 가져온다. (`Node.js`일 경우)
    ```ts
    import 'source-map-support/register'

    // or

    require('source-map-support').install()
    ```
4. 실행한다.



위 순서대로 적용 후 `dist/index.js` 파일을 실행하면 아래와 같이 에러가 발생한다.

```bash
Error: Error!
    at new Test (/Users/ukmo/Desktop/ts-error/index.ts:5:11)
    at Object.<anonymous> (/Users/ukmo/Desktop/ts-error/index.ts:9:14)
    ...
```

JavaScript파일을 실행했지만, 변환 전 TypeScript파일의 라인을 가르키고 있다.

꼭 `source-map-support` 모듈을 사용하지 않아도 된다. (이외에도 다양한 모듈이 있음)
