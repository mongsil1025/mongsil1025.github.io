---
title:  "[Java] SimpleDateFormat 으로 날짜 parsing 시, 오전/오후 시간 표기"
date: 2021-04-10
excerpt: ""
tags: [til, java, language]
categories: [til/language/java]
---

이력 발생 일자가 아래와 같이 14시 인데, 화면에서는 02시로 노출되는 이슈가 있었다.

``` sql
-- Oracle Data 데이터
|----------------------|
|HIST_OCC_DTS          |
|----------------------|
|2021-04-10 14:07:48 |
|2021-04-10 14:17:04 |
|----------------------|
-- 화면 노출 데이터
2021-04-10 02:07:48
2021-04-10 02:17:04
```

[https://help.gooddata.com/cloudconnect/manual/date-and-time-format.html](https://help.gooddata.com/cloudconnect/manual/date-and-time-format.html) 에 보면 SimpleDateFormat을 사용할 때, 날짜 패턴에 대한 내용을 볼 수 있다.

여기서 소문자 h는 Hour in am/pm (1-12)로, 오전/오후를 구분해서 순수 "시"를 반환한다.
대문자 H는 Hour in day (0-23)로, 오전/오후를 구분하지 않는다.

따라서, 위의 데이터처럼 동일하게 반환하기 위해서는 hh가 아니라 HH로 적어야한다!

``` java
public static void main(String[] args) {
  Date day = new Date();
  SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  System.out.println(format.format(day));
  // result : 2021-04-10 17:20:29

  format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
  System.out.println(format.format(day));
  // result 2021-04-10 05:20:29
}
```

**Table 28.3. Date Format Pattern Syntax (Java)**
|Letter|	Date or Time Component|	Presentation|	Examples
|:-------------|:------------------|:------|:------|
|G	|Era designator	|Text|	AD
|Y	|Year	|Year	|1996; 96
|M	|Month in year|	Month|	July; Jul; VII; 07; 7
|w	|Week in year|	Number|	27
|W	|Week in month|	Number|	2
|D	|Day in year|	Number|	189
|d	|Day in month|	Number|	10
|F	|Day of week in month|	Number|	2
|E	|Day in week|	Text|	Tuesday; Tue
|a	|Am/pm marker|	Text|	PM
|H	|Hour in day (0-23)|	Number|	0
|k	|Hour in day (1-24)|	Number|	24
|K	|Hour in am/pm (0-11)|	Number|	0
|h	|Hour in am/pm (1-12)|	Number|	12
|m	|Minute in hour|	Number|	30
|s	|Second in minute|	Number|	55
|S	|Millisecond|	Number|	970
|z	|Time zone|	General time zone|	Pacific Standard Time; PST; GMT-08:00
|Z	|Time zone|	RFC 822 time zone|	-0800
|'	|Escape for text/id|	Delimiter|	(none)
|''	|Single quote|	Literal|	'
