---
title:  "[Vue.js] v-validate 사용시 실시간으로 validate 안되는 현상"
date: 2020-11-18
excerpt: ""
tags: [til, vue]
categories: [til/vue]
---

### v-validate 사용시 실시간으로 validate 안되는 현상

사유 : `:value` 혹은 `v-value:` 를 사용해서 그렇다.
`v-model`을 사용했으면 항상 vue.$data 항목이 (console로 바꾸든, 인풋으로 바꾸든) 갱신된다.

하지만 `:value`는 `단방향 바인딩`이기 때문에 v-validate에서 유효성 검사에 실패한뒤, 다시 성공하는 케이스로 바꾸면 실시간으로 에러 알럿이 나타나지 않는다.

따라서, 실시간으로 유효성 검사를 하기 위해서는 `v-model` 로 체크할 값을 양방향 바인딩을 해주자.


``` html
// 단방향 바인딩
<input type="hidden" name="retShppcstId_<%=id%>" v-validate="{rules:{required:retShppcstIdRequired}}" :value="retShppcstInfo.shppcstId"/>

// 양방향 바인딩
<input type="hidden" name="retShppcstId_<%=id%>" v-validate="{rules:{required:retShppcstIdRequired}}" v-model="retShppcstInfo.shppcstId"/>
```
