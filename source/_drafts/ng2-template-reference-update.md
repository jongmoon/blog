---
title: angular2 에서 template reference 로 참조한 값이 갱신되지 않는 경우
tags:
---

## 문제 상황

아래와 같은 코드를 수행했을때,
```html
<select #mySelector>
  <option value="a">A</option>
  <option value="b">B</option>
</select>

<div>{{mySelector.value}}</div>
```
`div` 영역에는 의도한 A 값이 출력되지만, select 컨트롤에서 B 로 변경하면 반영되지 않습니다.
> angular 2.3.1 기준

## 해결 방법

[공식 문서](https://angular.io/docs/ts/latest/guide/template-syntax.html#!#ref-vars)와 [github 이슈](https://github.com/angular/angular.io/issues)에서 명확한 원인을 찾지 못해 구글링한 결과 # 를 사용해 reference 를 쓴다고 watching 이 되는 것이 아니라는 답변이 있었습니다.

[StackOverflow 답변](http://stackoverflow.com/questions/38335717/angular2-template-reference-variables-and-dynamic-updating)

DOM 을 watching 하기 위해 답변자가 제시한 방법은 다음과 같습니다.

  1. **ngModel** 혹은 **ngControl** 을 사용

```html
<select #mySelector [ngModel]="yourVar">
```

  2. **이벤트 핸들러** 적용

  이벤트 핸들러를 적용함으로서 watching 을 유발
  
```html
<select #mySelector (change)="yourEventHandler()">
```  

## 기타 

Template Reference 를 사용한다고 무조건 Watching 하도록 하면 불필요한 체크를 하게 되어 성능상의 이슈를 야기하기 때문인가?