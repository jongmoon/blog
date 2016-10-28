---
title: ngStyle 로 스타일이 반영되지 않아요.
date: 2016-10-27 23:01:54
tags:
---

## 상황

Angular2 에서는 element 개별 속성을 지정하기 위해서 다음과 같은 문법을 사용할 수 있습니다. (단위가 붙어 있으니 조금 이상하긴 합니다.)

```html
<div [style.left.px]="100"></div>
```

그러나 적용하고자 하는 속성이 여러개인 경우 [[ngStyle] 문법](https://angular.io/docs/ts/latest/guide/template-syntax.html#!#ngStyle)을 사용하게 되는데요.

다음과 같은식입니다.
**템플릿**
```html
<div [ngStyle]="setStyles()">
  absolute 로 지정한 DIV 영역
</div>
```

**컴포넌트 구현부**

```ts
setStyles() {
  let styles = {
    // CSS property names
    'left': '100',
    'top': '100px'
  };
  return styles;
}
```

`근데 이때 단위를 지정하지 않으면 속성이 반영되지 않는 이슈가 있습니다.`

위 예의 경우 left 속서 지정 시 `px`을 지정하지 않고 있는데요. 저 같은 경우엔 결과물에 아래와 같이 left 값이 노출되지 않아 원인 파악에 애를 먹었습니다.

**위 내용이 반영된 결과물**
```html
<div style="top:100px"></div> <!-- left 가 없어졌다!? -->
```

## 해결

반드시 단위를 지정해 줍니다.

```ts
setStyles() {
  let styles = {
    // CSS property names
    'left': '100px',/*단위를 붙여야 결과물에 속성이 노출됩니다.*/
    'top': '100px'
  };
  return styles;
}
```

> 단, html 에 doctype 을 지정하지 않아 quirks 모드로 동작하는 경우에는 단위를 제거해도 속성이 제대로 붙지만 권고사항은 아니므로 반드시 단위를 붙입니다.

## 회고

Angular2 를 하다보면 이렇게 에러나 경고 없이 의도하지 않은 동작을 보여주는 경우가 있는데요. 참 사용하기 쉬우면서도 이런 부분이 살짝살짝 발목을 잡네요.

차후에 이런 부분을 쉽게 디버깅할 수 있도록 Angular Core 쪽을 분석을 해봐야겠다는 생각을 합니다.

## 실습 예제
https://plnkr.co/edit/v6qgSfi349gMYzbCwROm?p=preview
