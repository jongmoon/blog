---
title: 360 뷰어에서 quaternion 의 활용
tags:
---

# Quaternion (사원수)

  - 복소수의 확장
  - 3차원상 점의 표기?
  - 방향의 판단은?
  
## 왜 필요한가?

  - 회전 변환 시 오일러각(서로 의존적인 x, y, z축의 각 표현방법)이 갖고 있는 문제를 해소한다.

### 오일러각의 문제

  1. 중간값 계산
  2. Gimbal Lock
   
## 주요 사용 기능

반드시 쿼터니언의 원리를 이해해야지만 사용할 수 있는 것은 아니다. 좀 더 직관적인 방식의 Euler 각(Angular) 표현과 Quaternion 표현 방식간의 전환이 가능하기 때문이다.

### 사전지식

**Normalize**

쿼터니언을 이루는 각 요소(실수)의 제곱값이 1 이 되도록하는 것이다.

이는 x, y 를 통해 이차원상의 단위원을 표현하기 위해 아래와 같은 공식을 쓰는 것처럼,
> x^2 * y^2 = 1 

단위 구(Sphere) 상의 점을 4 개의 점을 표기하는 것과 같은 개념이라고 보면된다.
> 왜 3개가 아니라 4개일까? 

두 개의 쿼터니언을 연산할 경우 동일한 기준을 마련해 두는 정도로 이해하면 좋겠다. 그래서 쿼터니언간의 계산에는 거의 항상 normalize 가 등장한다.


### 360 뷰어에서의 Quaternion 사용 사례

**FusionPoseSensor 에서 현재 회전 정보 반환**

위에서 언급한 오일러각의 회전 표현방식의 한계로 인해 회전 정보를 Quaternion 으로 변환하여 반환한다.

**회전한 상대값 계산**

FusionPoseSensor 에서 전달받은 쿼터니언 값으로 yaw, pich 값을 구한다.

임의의 단위 벡터값(0, 0, 1) 을 회전 전, 후 쿼터니언을 각각 적용하여 transform 하여 전, 후 시점의 벡터값을 구한다.
이후 이 두 벡터간의 각을 계산한다.(이 부분은 Quaternion 과 별개)

```js
vec3.transformQuat(prevPoint, prevPoint, prevQuaternion);
vec3.transformQuat(curPoint, curPoint, curQuaternion);		
```

위와 같은 연산을 pitchDelta, yawDeltaByRoll, yawDeltaByYa 값을 구하는데 각각 적용한다. 그리고 이 값은 최종 pitch, yaw 값을 계산하는데 사용한다.

```js
this.yaw = this.yaw + yawDeltaByRoll + yawDeltaByYaw;
this.pitch = this.pitch + pitchDelta;
```

두 벡터간의 사잇각을 크게 2가지 방향(pitch, yaw)으로 구한 후 이 각을 이용해 Quaternion 으로 변환한다.

```js
quat.rotateY(relativeQuaternionQ, relativeQuaternionQ, glMatrix.toRadian(this.yaw)); 
quat.rotateX(relativeQuaternionQ, relativeQuaternionQ, glMatrix.toRadian(this.pitch));
quat.copy(this.relativeQuaternion, relativeQuaternionQ);
```

컴포넌트는 이렇게 계산된 값을 이벤트 emit 한다.

## 정리

쿼터니언이 360Viewer 에서 어떻게 활용되는지를 확인해보았다.

360 에서 쿼터니언을 직접 컨트롤하는 영역은 `DeviceOrientationControl` 이 FusionPoseSensor 로부터 쿼터니언 값을 전달 받아 yaw, pitch, 그리고 상대적인 Quaternion 값을 구하는 _computeRelativeQuaternion 함수에 제한되었다고 봐도 무방하다.

그리고 Quaternion 을 가지고 직접 연산을 하기보다는 Quaterion 관련 메소드(normalize, transformQuat)를 활용하여 실제로 Quaternion 의 정체를 정확히 알지 못하더라도 문제를 해결하는데는 큰 지장이 없었다.

오히려 Quaternion 연산으로 이동한 벡터값을 가지고 각도를 구하는 부분들이 벡터 연산에 대한 이해를 필요로 하기 때문에 벡터 연산에 대한 이해가 더욱 요구된다 하겠다.


# 별개의 내용

각도를 알면 Quaternion 은 쉽게 구할 수 있기 때문에 Quaternion 에 대한 이해가 필수적이지는 않다. 단 Quaternion 에 대한 대략적인 느낌을 알면 사용하는데 큰 도움이 될 것 같다.

## x, y, z, w 

x,y,z 는 방향벡터, w 는 방향벡터를 기준축으로 한 각도라고 한다. 이 부분이 정말 사실인가 시각화해보고 싶다.

실제 위와 같은 방법으로 시각화 해보면 기대했던것과는 정말 다른 결과가 나온다.


## SLERP 의 이해

## Three 에서 제공하는 Quaternion 관련 메소드

### 차이 구하기

