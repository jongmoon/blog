---
title: Angular2 Quick Start 소스를 참고하여 Karma 테스팅 환경 설정해 보기
date: 2016-10-17 11:43:26
tags:
---

## 상황

angular2 에서는 cli(Command Line Interface)를 통해 Karma 테스트 환경을 설정할 수 있습니다. 그러나 Karam 가 포함되지 않은 비 CLI 기반 angular2 개발환경에서(eg. [angular2 seed](https://github.com/angular/angular2-seed), [universal](https://github.com/angular/universal))는 Karma 환경을 구축하기 위해서는 귀찮은 작업들을 해주어야 합니다.

저 역시 angular2 cli 기반이 아닌 angular2 프로젝트(TypeScript)에 karma 를 적용해 보고자 했으나...

  - Angular 홈페이지에서 제공하는 [quickstart](https://github.com/angular/quickstart)에 testing 코드를 그대로 적용하면 제대로 안되고
  - [Testing 튜토리얼](https://angular.io/docs/ts/latest/guide/testing.html)의 Setup 부분을 봐도 자세히 언급은 되지 않았습니다.

그래서 테스팅 환경으로 karma 가 포함되지 않은 angular2 프로젝트 기반에서 quickstart 에 있는 karma 테스팅 환경 코드를 가져다 쓰며 삽질했던 경험을 올려봅니다.

## 설정 과정
### 1. 기본 환경 설정

1. quickstart 의 package.json 를 수정하고
2. 테스트 환경 세팅 파일을 작업 중인 프로젝트에 복사했습니다.

**package.json 수정**
```json
"scripts": {
    /*... */
    //tsc 는 ts -> js 트랜스파일러 (-w 는 watch 모드) 실행 명령
    //concurrently 는 동시에 실행을 가능하게 하는 명령어
    "test": "tsc && concurrently \"tsc -w\" \"karma start karma.conf.js\"",
    "test-once": "tsc && karma start karma.conf.js --single-run",
    "tsc": "tsc",
    "tsc:w": "tsc -w"
},
"dependencies": {
    /* ...*/
    "systemjs": "0.19.38"
},
"devDependencies": {
    /*... */
    /*  */
    "concurrently": "^2.2.0",
    /* Jasmine 을 Testframework 으로 사용한다.*/
    "jasmine-core": "~2.5.2",
    /* Karma 관련 필요한 라이브러리들을 명시한다.*/
    "karma": "^1.3.0",
    "karma-chrome-launcher": "^2.0.0",
    "karma-cli": "^1.0.1",
    "karma-htmlfile-reporter": "^0.3.4",
    "karma-jasmine": "^1.0.2",
    "karma-jasmine-html-reporter": "^0.2.2",
    "karma-phantomjs-launcher": "^1.0.2",
    "karma-sourcemap-loader": "^0.3.7",
    "karma-webpack": "^1.8.0"
}
```

> package.json 을 수정했으므로 npm install 을 해주어야 위에 설정된 npm 모듈들이 다운로드 받아지므로 잊지 말아주세요.

**환경세팅 파일들 복사**

프로젝트 최상단에 다음 파일들을 복사 
> 경로는 원하는 곳으로 지정합니다. 아래 파일들을 참조할 수 있기만 하면 되니까.

  - **karma.conf.js**: karma 환경설정 파일
  - **karma-test-shim.js**: 앵귤러 테스팅 환경을 준비하고 카르마를 띄우기 위한 파일(여기서 system.config.js 로 로딩함)
  - **systemjs.config.js**: 애플리케이션과 테스트할 파일을 로딩
  - **systemjs.config.extra.js**: quickstart 프로젝트에도 해당 파일을 참조하고 있지만 실제는 존재하지 않는 파일이므로 무시

### 2. karma.conf.js 의 수정

Karma 에서 로딩할 파일을 지정합니다.

angular2 quickstart 에서는 `/app 폴더` 하위에 ts 파일 및 js 파일, test 파일(*.spec.tx, *.spec.js)이 통째로 들어가기때문에 아래와 같이 설정되어 있습니다.

```js
  var appBase    = 'app/';       // transpiled app JS and map files
  var appSrcBase = 'app/';       // app source TS files

  config.set({
    /* 위 경로는 아래와 같이 karma 설정에서 url 패턴으로 사용됩니다. */
    files: [
      {pattern: appBase + '**/*.js', included: false, watched: true},
      {pattern: appSrcBase + '**/*.ts', included: false, watched: false},
      {pattern: appBase + '**/*.js.map', included: false, watched: false}
    ]
  })
```

그러나 저의 경우에는 위와 다른 경로로 사용할 예정입니다.

```js
  var appBase    = 'dist/app/';
  var appSrcBase = 'src/app/';
```

아마 저랑 비슷한 경험을 한 부분은 여기까지는 별 문제가 없을 것 같습니다.그럼 여기까지 따라하면 잘 될까요?

확인을 위해 테스트하려는 컴포넌트가 있는 경로에 spec 파일을 추가해 봅니다.(참고: [spec파일이 뭔지 모르면 클릭](https://angular.io/docs/ts/latest/guide/testing.html#!#1st-karma-test))
```ts
describe('1st tests', () => {
  it('true is true', () => expect(true).toBe(true));
});
```

`npm test` 를 실행하면 아마 describe 와 it 심볼을 찾지 못할 것입니다.

```
error TS2304: Cannot find name 'describe'.
error TS2304: Cannot find name 'it'.
error TS2304: Cannot find name 'expect'.
```

### 3. tsconfig.json 에서 jasmine 타입 추가

tsconfig.json 에 jasmine 에서 사용하는 심볼을 찾을 수 있도록 다음과 같이 타입을 지정해줍니다.

```json
"types": [
  /* 생략 */
  "jasmine"
]
```

`npm test` 를 실행해 봅니다.

이제 에러가 발생하지 않습니다. 근데 위해서 추가한 test 케이스는 실행되지 않을 것입니다.

왜 그럴까요? 다음 항목에서 원인을 찾을 수 있습니다.

### 4. karma-test-shim.js 에서 builtPath 변경

QuickStart 팩에서 사용중인 karma-test-shim.js 는 다음과 같이 build 된 경로를 체크하는 로직을 체크합니다. 

```js
var builtPath = '/base/app/';

function isBuiltFile(path) {
  return isJsFile(path) && (path.substr(0, builtPath.length) == builtPath);
}
```

그러나 1번 과정(`1. 기본 환경 설정`)에서 배포 경로를 바꾸었으므로 importing 목록에 포함되지 않습니다. 따라서 저의 경우는 다음과 같이 변경했습니다.

```js
var builtPath = '/base/dist/app/';
```


> [참고](https://github.com/karma-runner/karma/issues/1607) Karma 는 로딩파일을 base 폴더 하위에 로딩합니다. 그래서 karma 를 통해 서버가 구동된 후 브라우저에서는 관련 파일을 base 폴더 하위에서 접근하게 됩니다.
> 

자, 'npm test' 하면 다음과 같이 테스트가 수행된 것을 확인할 수 있습니다.

```
....Executed 1 of 1 SUCCESS (0.008 secs / 0.002 secs)
```

## 얻은 경험

길게 풀어서 정리해보았지만 여기서 얻었던 경험은 다음과 같습니다.

  * karma 는 자체가 Test Framework 이 아니라 jasmine, mocha 와 같은 Test Framework 기반으로 만든 `테스트 코드를 구동할 수 있는 환경`을 만드는 도구이다. (말이 좀 어렵나...)
  * karma 는 base 폴더 하위에 모듈 로딩을 한다. 
      - 중간 중간에 base 라는 이름을 자주 볼 수 있는 이유다.
      - [Get rid of the "base" directory](https://github.com/karma-runner/karma/issues/1607)
  * tsconfig.json 을 통해 jasmine 관련 심볼을 찾을 수 있도록 타입을 추가할 수 있다.

## 결론

이미 karma 와 systemjs 에 익숙한 사용자는 아마 이 문서를 찾아 볼 일은 없을 것 같습니다.  

하나씩 삽질하는 것을 차근 차근 정리하다 보면 문제가 해결되는 경우가 많아 보통 원노트(One Note)를 이용해 정리하는데 이번에는 블로그를 연 김에 이곳에서 정리하여 올려둡니다.

저 같이 karma 나 systemjs 관련 경험이 없는 상태에서 angular2 quickstart 기반의 테스팅 환경을 가져다 쓰시려는 분께 참고가 되면 좋겠습니다.


