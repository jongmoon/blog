---
title: ng2-change-detection-strategy-onpush
tags: angular2 change detection onpush
---

##Change Detection 전략 - OnPush

특징
  - @Input()으로 바인딩된 데이터 변경 시 OnChange 를 유발한다는 점에서는 Default 전략과 동일하지만,
  - 변경 사항이 있다고 판단하는 경우에만 하위 Tree 체크
    - Default 는 변경되지 않았다고 하더라도 하위 Tree 를 체크함
    - 따라서 Immutable 로 하지 않으면 하위 노드로의 갱신이 이루어지지 않을 수 있음.