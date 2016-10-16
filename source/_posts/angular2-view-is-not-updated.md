---
title: Angular2 에서 View 가 갱신되지 않아요
date: 2016-10-12 21:41:52
tags:
---

## 이슈
Model(data)를 분명히! 갱신했음에도 불구하고 해당 데이터가 바인딩된 View 가 갱신되지 않았습니다. 레퍼런스나 값을 변경하면 이 똑똑한 Angular2 가 알아서 화면에 그려주는데 당췌 갱신하질 않습니다.

>원인을 파악하기 위해 다음과 같이 상황을 단순화 시켜보았습니다.

## 상황
myService 에는 다음과 같은 데이터를 제공합니다.

```ts
public keywords:Array<string> = ["keyword1", "keyword2", "keyword3"];
public values:any = {
  "keyword1": "first",
  "keyword2": "second",
  "keyword3": "third"
}
```

View 에서는 keywords 를 순환하며 각 key 에 해당하는 values 를 화면에 노출합니다. 본 예제에서는 values 의 key 에 해당하는 값을 추출하기 위해 getPropertyValue 라는 `pure 한 파이프`를 사용합니다.(물론 굳이 그럴 필요는 없지만 그렇게 적용했습니다.)

```html
<table>
  <tbody>
    <tr>
      <th>key</th>
      <th>value</th>
    </tr>
    <tr *ngFor="let keyword of myService.keywords">
      <td>
        <strong>{{keyword}}</strong>
      </td>
      <td>
        <strong>{{myService.values | getPropertyValue:keyword}}</strong>
      </td>
    </tr>
  </tbody>
</table>
```

## 문제 재현

문제는 values 의 값을 변경할 때 발생합니다.

임의로 다음과 같이 값을 변경해 봅니다.
```ts
values['keyword3'] = "UPDATED";
```

분명 keyword3을 키로 갖는 values 값이 UPDATED 로 변경되었습니다. 그러나 화면에 갱신이 되지 않습니다.

## 원인

처음에는 변경한 대상이 ngFor 에서 iteration 하고 있는 대상인 keywords 이 아니라 values 이기 때문이라고 오해했습니다만 실상은 그렇지 않았습니다.

getPropertyValue 라는 `pure한 파이프`가 문제였습니다.

`ngFor` 처럼 Object 의 레퍼런스 혹은 그 레퍼런스에 속한 값이 변경된 경우 변경을 감지할 것이라는 기대를 했었건만... `pure 한 파이프`는 그렇게 동작하지 않았습니다.

파이프가 바라보던 myService.values 의 레퍼런스가 변경되지 않았기 때문입니다. values 의 keyword3 속성의 값은 분명 변경했지만 values 자체의 레퍼런스는 바뀌지 않은 것입니다.

pure 한 Pipe 는 자신이 연결된 값을 비교(===)하여 변경되지 않으면 파이프를 수행하지 않습니다. (https://angular.io/docs/ts/latest/guide/pipes.html 페이지에서 Pure 와 Impure Pipes 참고)

## 해결방법

다음 중 어느 방법으로도 해결할 수 있습니다.

**1. 파이프를 대체할 수 있는 경우 파이프를 사용하지 않습니다.**

위 상황의 경우 굳이 파이프를 쓸 필요가 없습니다. 아래와 같이 [keyword] 로 변경합니다. 
```html
<strong>{{myService.values[keyword]}}</strong>
```

**2. values 의 레퍼런스를 변경합니다.**

만약 파이프를 사용해야 하는 경우라면 values 의 레퍼런스를 변경합니다.
```ts
values = Object.assign({}, values);
```


**3. Pipe 를 Impure 로 생성합니다.**

기존 Pipe 에서 pure 속성을 false 로 변경해주면 Impure 하게 변경됩니다.

```ts
@Pipe({
  name: 'getPropertyValue',
  pure: false
})

export class MyPipe {
  transform(obj, prop) {
    return obj[prop];
  }
}
```

그러나 이 방식은 값이 변경여부를 체크하지 않고 항상 파이프를 수행하기 때문에 대부분의 상황에서는 성능상 이슈가 생길 수 있습니다.

## 인사이트

파이프를 Primitive 타입이 아닌 Object 에 적용하는 경우 주의가 필요하겠습니다.

## 확인 가능한 예제 
https://plnkr.co/edit/Pf7Euer6dXiAnlULAQhw?p=preview




