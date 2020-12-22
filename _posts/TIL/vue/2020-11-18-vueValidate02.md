---
title:  "[Vue.js] v-validate.continues"
date: 2020-11-18
excerpt: ""
tags: [til, vue]
categories: [til/vue]
---

### v-validate 의 `continues` 옵션

``` html
v-validate="'required|email'"
```
위와 같이 유효성 검사할 항목이 N개일 경우, `required`에서 실패하면 뒤에 rules를 검사하지 않는다.

<div class="alert alert-primary">
  By default vee-validate uses a fastExit strategy when testing the field rules, meaning when the first rule fails it will stop and will not test the rest of the rules. You can use the .continues modifier to force this behavior to test all rules regardless of their result.
</div>

<i class="icon icon-link" style="display: inline-block;"></i>[Vue 공식문서](https://vee-validate.logaretm.com/v2/api/directive.html#persist)

따라서, `v-validate.continues`와 같이 기입해줘야 한다.
