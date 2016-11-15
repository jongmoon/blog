---
title: 간단한 Reactive UI 의 Angular2, React 구현 비교 
date: 2016-11-14 22:02:35
tags: ng2, angular2, react
---

## 프롤로그

몇 년전 `자동차 크기 비교`를 시각적으로 해보고 싶은 마음에 http://moonserver.cafe24.com 를 만들어 보았습니다. (다만 호스팅이 머지않아 만료될 예정입니다. 어떻게 할지는 고민해 보려고 합니다.)

![](car-vs.gif)

## 구조

### 마크업

```html
<div id="stage">
    <div class="cube">
        <div class="plane one"></div>
        <div class="plane two"></div>
        <div class="plane three"></div>
        <div class="plane four"></div>
        <div class="plane five"></div>
        <div class="plane six"></div>
    </div>
</div>
```

### 코드

```js
stage.style.transform = "rotateY(" + currDeg + "deg)";
```

## 시도

(유행에 편승해 블로그 방문자를 늘려 보려는 것은 아니구요 ^^;;)

문뜩 최근 각광받는 프레임워크들을 이용해 비교해 보면 유용할 것같다는 생각이 들어 최근 Angular2 를 사용해본 김에 React 와 Angular2 를 기반으로 간단히 만들고 비교해 보았습니다.

## 알아보고 싶었던 것

* 어떤 Framework 이 더 빠를까?
* 과연 Framework 를 쓰면 `성능`이 빨라질까 혹은 느려질까?
* 특히 스크롤이 생기는 부분에서 `버벅임`이 개선될 수 있을까?
    - 이를 위해 성능저하를 유발하는 배경지정을 해두었습니다. 

> 여기서는 로딩 성능이나 파일 크기등의 비교는 제외하였습니다.

## 어떻게 적용했나?

`TODO: github 에 반영 예정`

어떤 스타일로 적용했는지 맛만 보기 위해 그냥 코드 일부만 발췌했습니다. 

  - Angular2 는 `View` 와 View 에 Data 를 제공하고 View 를 제어하는 부분이 독립적으로 분리 가능한 형태입니다.
  - 반면 React 는 `View` 가 `jsx 파일`에 View 관련 제어 코드와 함께 녹여집니다.

### Angular2

**HTML**
```html
<!-- compare-view.component.html -->
<div id="wrapper">
  <div id="container"
      (touchstart)="handleTouchStart($event)"
      (touchmove)="handleTouchMove($event)"
      (touchend)="handleTouchEnd($event)">
        <div id="stage" [ngStyle]="setStyles()"> 
          <app-cube *ngFor="let item of cubeInfoList" [cubeInfo]="item"></app-cube>
        </div>
      </div>
</div>

<!-- cube.component.html -->
<div class="cube backfaces" [ngStyle]="setBackfacesStyles()">
  <div class="plane one" [ngStyle]="setPlaneOneStyles()"></div>
  <div class="plane two" [ngStyle]="setPlaneTwoStyles()" [class.car_left_full]="!cubeInfo.isBiggest"></div>
  <div class="plane three" [ngStyle]="setPlaneThreeStyles()"></div>
  <div class="plane four" [ngStyle]="setPlaneFourStyles()"></div>
  <div class="plane five" [ngStyle]="setPlaneFiveStyles()"></div>
  <div class="plane six" [ngStyle]="setPlaneSixStyles()"></div>
  <app-size-indicator [cubeInfo]="cubeInfo"></app-size-indicator>
</div>
```

**SCRIPT**
```ts
//compare-view.component.ts
setStyles() {
  let style = {
    'width': this.width + 'px', 
    'left': this.left + 'px',
    'transform': "rotateY(" + this.roateDeg + "deg)"
  }
  return style;
}

//cube.component.ts
setBackfacesStyles() {
  return {
    'left': this.cubeInfo.pos.x + "px",
    'top': this.cubeInfo.pos.y + "px",
    'transform': "translateZ(" + this.cubeInfo.pos.z + "px)",
    'width': this.cubeInfo.size.x,
    'height': this.cubeInfo.size.y,
    'borderColor': this.cubeInfo.color
  }
}

setPlaneOneStyles() {
  return {
    "top" : (this.cubeInfo.size.y - this.cubeInfo.size.z) / 2 + "px",
    "width" : this.cubeInfo.size.x + "px",
    "height" : this.cubeInfo.size.z + "px",
    "transform" : this.getTransformCSS(0, this.cubeInfo.size.y / 2),
    "borderColor": this.cubeInfo.color
  }
}
//.. 이하 생략
```

### React

**JSX**
```jsx
//CompareView.jsx
// ... 상단 생략 ...
render: function() {
  var cubes = this.cubeInfoList && this.cubeInfoList.map(function(info, index) {
    return (
      <Cube key={index} cubeInfo={info}/>
    )
  });

  return (
    <div id="container"
      onTouchStart={this.handleTouchStart}
      onTouchMove={this.handleTouchMove}
      onTouchEnd={this.handleTouchEnd}>
      <div id="stage" style={this.setStyles()}>
        {cubes}
      </div>
    </div>
  );
},

setStyles: function() {
  let style = {
    'width': this.width + 'px', 
    'left': this.left + 'px',
    'transform': "rotateY(" + this.roateDeg + "deg)"
    }
  return style;
},

//Cube.jsx
render: function() {
  var cube = this.props.cubeInfo;
  
  return (
    <div className="cube backfaces" style={this.setBackfacesStyles()}>
      <div className="plane one" style={this.setPlaneOneStyles}></div>
      <div className={cube.isBiggest ? "plane two" : "plane two car_left_full"} style={this.setPlaneTwoStyles()}></div>
      <div className="plane three" style={this.setPlaneThreeStyles()}></div>
      <div className="plane four" style={this.setPlaneFourStyles()}></div>
      <div className="plane five" style={this.setPlaneFiveStyles()}></div>
      <div className="plane six" style={this.setPlaneSixStyles()}></div>
      <SizeIndicator color={cube.color} isBottom={cube.isBiggest} width={cube.size.x} height={cube.size.y} depth={cube.size.z} marks={cube.marks} diff={cube.diff}/>
    </div>
  );
},
// ... 하단 생략 ...
```


## 결과

### 1차 시도(약간 삽질에 대한 잡설)

체감 성능: Angular2 > React ~= Vanilla ???!

> Angular2 가 버벅임을 해결했다!? (삑!!~~~ No) 

**비밀은 minimum-scale=1.0 지정시 성능 저하문제**

현재 노트 4(Android 6.0.1) 기본 브라우저(삼성인터넷 4.0.20-65)에서는 헤더에 `minimum-scale=1.0` 이라고 메타 정보를 입력한 경우. `컨텐츠가 길어져 스크롤이 생기는 시점`에 엄청난 성능 저하가 발생합니다.

```html
<meta name="viewport" content="width=device-width, initial-scale=1, minimum-scale=1">
```

![](minimum-scale-bug.png)
첫 시도 시 Angular2 의 HTML 에만 위 옵션을 지정하지 않았습니다. 그래놓고는 Angular2 가 빠르다고 신기해하고 있었죠. 

그런데 뭔가 석연치 않아 어디가 원인인지 하나씩 찾아가는 중에 발견한 부분입니다.

### 2차 시도

1. minimum-scale 은 모두 제거
2. 삼성브라우저 대신에 chrome 모바일 버전으로 비교
  - 혹시나 모를 브라우저 잠재적 버그에 대한 대응?!
<!-- 3. 또한 각각 플랫폼 별로 raf 가 적용된것과 적용되지 않은 것을 비교하였습니다. -->

#### 결과

사용자의 Touch 반응에 의해 즉각적으로 움직여야 하므로 반응성 부분이 중요한 부분입니다. 

> 그러나 실제 프레임워크 간의 프레임 차이가 두드러져 보이지 않았습니다.

그래도 뭔가 차이를 찾아보고 싶어 Chrome 개발자 도구의 `Timeline` 을 열었습니다.

**1. CPU 점유율에서 다음과 같은 차이가 있었는데요.**

`(낮을 수록 좋음)`
**vanilla** << **angular2** < **react**

**2. 메모리 점유율도 다음과 같았습니다.**

`(낮을 수록 좋음)`
**vanilla** << **angular2** ~~ **react** (` ~~ `: 비슷)


**스크린샷(터치에 의한 회전정보 변경)**
왼쪽부터: `Vanilla -> React -> Angular2`
![](touch-version-compare.png)


추가적으로 `requestAnimationFrame(RAF)` 에 의해 DOM 정보를 갱신하는 방식에 대한 비교도 추가하였습니다.

그러나 아직 아래 프레임워크에 대한 경험이 많지 않아 각 `프레임워크 Way`를 따랐는지 몰라 일단 참고만 해주시면 좋겠습니다.()
>  
>  1. React 는 직접 DOM 을 제어하지 않고 setState 를 통해 상태를 변경하는 방식을 취하였습니다
>  2. Angular2 는 아래와 같은 방식으로 사용했습니다.
>  

```ts
updateRotationDegV2() {
  this.ngZone.runOutsideAngular(() => {
    requestAnimationFrame(() => {
      if (this.stageEl) {
        this.renderer.setElementStyle(this.stageEl, 'transform', 'rotateY(' + this.roateDeg + 'deg)');
      }

      //터치 중이면 RAF 를 통해 다시 frame 갱신
      if (this.isTouching) {
        this.updateRotationDegV2();
      }
    });
  });
}
```

**스크린샷(RAF 에 의한 회전정보 변경)**
![](raf-version-compare.png)

> 단, 위 정보의 세부 수치들이 매 테스트마다 조금씩 달라지므로, 절대적인 의미를 두지마시고, 대략적으로 이런 상대적인 차이를 보이는 구나 정도의 판단 근거로 봐주셨으면 합니다.

**각 프레임별 프레임워크 호출 비용**

위에서 React 대비 Angular2 가 스크립트 비용이 좀더 적게 들었는데요. 각 프레임별로 확대해 비교해 보면 대략 아래와 같은 차이를 보입니다. 

![](frame-call-cost.png)

> touchmove 시점에 각 프레임워크에서 변화를 체크하고 DOM 에 반영하는 함수들이 호출되고 있는데요. Angular2 가 근소한 차이로 스크립트 비용이 적게 발생하고 있습니다.
> 
> 한 프레임당 차지하는 프레임워크에 대한 비용 차이(노트4 기준)가 0 ~ 2ms 정도 되는 것 같습니다.



## 각 프레임워크에 대해 느낀 장단점

CPU 점유율 차이에 따른 모바일 배터리 소모가 조금 달라질 수는 있겠지만, 두 프레임워크 내에서 한 프레임당 차지하는 프레임워크에 대한 비용 차이가 절대적으로 큰 것은 아니기 때문에 이를 가지고 어떤 것이 성능이 좋다/나쁘다고 판단하는 것은 무리가 있을 거 같습니다. 중요한 것은 프레임 손실율이니까요.

### Angular 장/단

**장**
  - **View 코드의 명확한 분리가 가능**
      +  관점의 전환 -> View 는 진짜 View 에만 포커싱, View 에 바인딩된 데이터만 변경해 놓으면 알아서 갱신되는 방식
  - **편리한 CLI**
  - **TypeScript**

 
**단**
  - **간단한 테스트 코드를 작성 불편**
    + cli 와 plukr.co 로 커버되긴 하지만 기존 JS 개발자 입장에서 불편
  - **점점 많아지는 코드들...**

### React 장/단
**장**
  - **트랜스파일러 환경구축없이 바로 실행해보기에 더 편함**

**단**
  - **JSX - 마크업과 JavaScript 의 모호한 경계**
    + 예전에는 JS 개발상 어쩔 수 없다고 생각했는데, 트랜스파일 환경이 일반화된 요즘은...

## 마지막 의견

프레임워크에 대한 경험이 부족한 상태에서 비교한 것이라 결과물 또한 초보적인 수준이라고 생각이 됩니다.

일단 체감적인 성능차이(FPS)는 크지 않았습니다. 일단 Vanilla 구현을 제외하고 얘기를 하면 React 가 Angular 대비 다소 높은 CPU 점유율을 보여주지만 성능에 큰 영향을 줄 정도는 아니었습니다.

두 프레임워크 모두 진화하는 중이며 성능은 지금보다 더 결국 상향 평준화 될 것으로 기대되고 어쩌며 이런 비교는 무의미할 수도 있을 거 같기도 합니다. (*그러나 눈으로 확인해야 직성이 풀리니 이런 비교는 계속 하고 싶습니다.*)

결국 어떤 프레임워크가 나 혹은 우리에게 적합한 방식인지가 중요한 것 같습니다. 저의 경우에는 Angular2 가 data 와 View 를 좀 더 명확하게 분리하려고 한 느낌을 받았습니다. 그래서 관심사에만 더 집중할 수 있게 해준 Angular2 방식이 좀더 마음에 끌립니다.


## 기타

  * 어떤 이유 때문에 스크립트 비용 차이가 있는걸까 ? 비교 방식의 차이라고 추측은 하지만 그 추측이 맞나?

