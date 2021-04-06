---
title:  "[Java] Object field 값의 Not Null 체크"
date: 2021-04-06
excerpt: ""
tags: [til, java, language]
categories: [til/language/java]
---

아래와 같은 객체가 있을 때, 내부 모든 Field에 대해 NotNull 체크를 하기 위해 어떤 방법이 제일 최선일까 고민을 했습니다.

``` java
@Getter @Setter @Builder
public static class OrdShppcstApiParam{

  private String itemId;
  private String itemRegDivCd;
  private Long minOnetOrdPsblQty;
  private BigDecimal sellprc;
  private String shppMainCd;
  private String shppMthdCd;
  private String siteNo;
  private String splVenId;

  private String deliPsblYn;
  private String whoutShppcstId;
}
```

## 기본적인 방법 - StringUtils.notBlank

``` java
if(StringUtils.isNotBlank(param.getItemId()) && StringUtils.isNotBlank(param.getItemRegDivCd) ...) {}
```

if 문에 모든 필드를 체크하는 것

- 모든 필드를 수기로 작성해줘야 함

## 기본적인 방법 - Reflection으로 모든 필드를 Get해서 체크

``` java
for (Field f : obj.getClass().getFields()) {
  f.setAccessible(true);
  if (f.get(obj) == null) {
     return false; // 하나라도 null 이면
  }
}
```

리플렉션으로 모든 필드를 체크하는 방법

## Stream을 사용하는 방법

``` java
boolean checkNotNull = Stream.of(param.getItemId(), param.getItemRegDivCd(), ...).allMatch(Objects::nonNull);
```

스트림과 람다를 사용하면 보다 깔끔하게 구현할 수 있다.
- 특정 필드만 체크하도록 구현할 수 있다
- 대신, 모든 필드를 개발자가 넣어줘야 하는 건 첫번째 방법과 동일하다.

## 참고문서
https://stackoverflow.com/questions/12362212/what-is-the-best-way-to-know-if-all-the-variables-in-a-class-are-null/42188504
