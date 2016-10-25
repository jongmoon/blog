---
title: position absolute 와 doctype 에 따른 height 이슈
date: 2016-10-25 23:03:47
tags: HTML, position:absolute, DOCTYPE
---

## 이슈

### 퀴즈1

아래 마크업의 `.parent` div 의 height 값은 얼마일까요?

```html
<body>
    <div class="parent">
        <div class="child"></div>  
    </div>
</body>
```

```css
.parent {
}

.child {
  position: absolute;
  width:100px;
  height:100px;
}
```

**정답**

`.parent` 의 높이값은 0

**이유** 

element 의 position 이 absoulte 인 경우 부모 element 크기에 영향을 주지 않기 때문입니다.
  - http://stackoverflow.com/questions/12070759/make-absolute-positioned-div-expand-parent-div-height
  - https://opentutorials.org/course/45/60

### 퀴즈2

아래 마크업에서 `.content` 의 height 값은 얼마일까요? (화면크기가 100px 이라고 가정)

```html
<body>
    <div class="content"></div>
</body>
```

```css
body {
  margin: 0px;
}

.content {
  height: 50%;
}
```

**결과**

화면크기가 100px 이므로 body 도 묵시적으로 100px 이므로 자식인 .content 영역은 50px 일 것이라고 생각했지만!!

`DOCTYPE` 지정여부에 따라 결과가 달라집니다.

#### 1) DOCTYPE 미지정시

`.content` 의 height 값은 50px 이 됩니다.

**이유**
미지정 시 [Quirk 모드](https://developer.mozilla.org/ko/docs/호환_모드와_표준_모드)로 동작하는데 이 모드에서는 추정되는 부모 element 높이를 기준으로 height 의 % 를 계산합니다.

한편,
#### 2) 아래와 같이 DOCTYPE 지정시
```html
<!DOCTYPE html>
```

`.content` 의 height 는 0 입니다.

**이유**

HTML5 spec 에서는 element 의 height 에 % 지정시,
>  1. 자신을 포함한(부모) element의 높이가 명시적으로 도출할 수 있는 경우 해당 높이 값에 대한 비율(%)을 계산합니다.
>  2. 부모의 높이가 명시되지 않은 경우 자식(Content)의 height 를 따릅니다.

[참고: height 에 퍼센티지(%) 지정시 처리 방식](http://stackoverflow.com/questions/5657964/css-why-doesn-t-percentage-height-work)

위 경우에는 `.content` 의 부모가 없어 부모 높이를 구할 수 없고, 자식 컨텐츠도 포함하지 않기 때문에 0 이 되는 겁니다.

그러나 W3C 에서는 HTML5 표준을 따르기 위해서는 반드시 DOCTYPE 을 붙이도록 권고하고 있습니다. (https://www.w3.org/QA/Tips/Doctype) 

## 회고

그 동안 다음 내용을 잘 인지하지 못하고 있던 탓에 다음 내용이 섞인 이슈 해결에 애를 먹었는데요. 이번 기회에 확실히 숙지하도록 해야겠습니다.

  * `position:absolute` 는 부모 크기에 영향을 주지 않는다는 점
  * height 를 비율로 지정한 경우 `DOCTYPE` 여부에 따른 동작 방식

