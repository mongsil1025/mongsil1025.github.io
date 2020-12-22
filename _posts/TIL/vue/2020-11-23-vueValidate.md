---
title:  "[Vue.js] v-validate Disable 된 항목에서 Validation 안됨"
date: 2020-11-23
tags: [til, vue]
categories: [til/vue]
breadcrumbs: true
---

v-validate는 기본적으로 `disabled` 된 항목에서는 validation을 pass 합니다.

1. `disabled` 된 항목도 validate 하고 싶다면 `useConstraintAttrs` 옵션을 추가하면 됩니다.

``` javascript
import Vue from 'vue';
import VeeValidate from 'vee-validate';

const config = {
  useConstraintAttrs: true
};

Vue.use(VeeValidate, config);
```

<i class="icon icon-link" style="display: inline-block;"></i>[Vue.js 공식문서](https://vee-validate.logaretm.com/v2/configuration.html)

2. `disabled` 항목을 `readonly` 로 변경

`readonly` 항목을 validate 하기 때문에, `disabled` -> `readonly`로 변경하는 것도 방법입니다.

<i class="icon icon-link" style="display: inline-block;"></i>[Github Discussion](https://github.com/logaretm/vee-validate/issues/1093)
