---
title:  "[Java] SimpleDateFormat 오류 - For input string, multiple points"
date: 2021-05-12
excerpt: ""
tags: [til, java, language]
categories: [til/language/java]
---

서버에서 간헐적으로 아래 에러가 나는 것을 확인했습니다.

``` console
For input string: ".373373EE22"
multiple points
For input string: "1122E.11222E"
```

1. SimpleDateFormat is not thread safe

구글링 해서 아래 글을 보았고 해결방법을 찾았습니다.
`SimpleDateFormat` 은 스레드 세이프하지 않으므로 `static final`로 정의해서 공유해서 쓰면 안되고, 새로 만들어서 써야 합니다.

[잘못된 예]
``` java
static final SimpleDateFormat F = new SimpleDateFormat("yyyyMMdd");
public static parseDate(String sDate){
    return F.parse(sDate); //throws a NumberFormatException
}
```

[올바른 예]
``` java
public static parseDate(String sDate){
    SimpleDateFormat F = new SimpleDateFormat("yyyyMMdd");
    return F.parse(sDate);
}
```

참고 : [https://medium.com/@daveford/numberformatexception-multiple-points-when-parsing-date-650baa6829b6](https://medium.com/@daveford/numberformatexception-multiple-points-when-parsing-date-650baa6829b6)

2. [SimpleDateFormat Javadoc 문서](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html)

문서에 `Synchronization` 문단을 보면 아래와 같은 내용이 있습니다.

```
Date formats are not synchronized. It is recommended to create separate format instances for each thread. If multiple threads access a format concurrently, it must be synchronized externally.
```

`SimpleDateFormat`은 `DateFormat`을 extend 한 클래스로, `DateFormat`을 extend 한 클래스들은 모두 thread safe하지 않으니 주의가 필요할 것 같습니다.
