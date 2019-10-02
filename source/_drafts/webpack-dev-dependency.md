---
title: webpack-dev-dependency
tags:
---

## webpack 에서 import 가 안되는 경우

### 현상

global npm 모듈로 `webpack` 이 설치되어 있음에도 불구하고, 아래와 같이 jquery 구문을 import 하는 경우 webpack 수행 시 jquery 가 import 되지 않는다.

### 원인

[webpack-Getting Started](https://webpack.js.org/guides/get-started/)의 스텝 중 하나로 webpack 을 인스톨하는 과정이 있다.

그 중 아래와 같이 로컬 영역에 webpack 을 install 하지 않았다.

```
npm install --save-dev webpack
```

### 해결
이미 global 로 설치된 상황이었기 때문에 설치하지 않았지만, 문제 해결을 위해서는 반드시 필요한 과정으로 판단된다.

### 궁금증

  - --save-dev 를 하면 package.json 의 devDependency 에 포함이 된다. dependencies 와의 차이점은 뭔가?
