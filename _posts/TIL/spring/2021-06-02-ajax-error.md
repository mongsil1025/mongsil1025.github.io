---
title:  "[Spring MVC] Ajax 콜 Unexpected end of JSON input 에러"
date: 2021-06-02
excerpt: ""
tags: [til, spring]
classes: wide
categories: [til/spring]
---

``` java
@RequestMapping(value = "/getEmartOfflineRepItemInfo", method = RequestMethod.GET)
  public @ResponseBody TenantRepItemDto getEmartOfflineRepItemInfo(@RequestParam("venId") String venId) {
    if(StringUtils.isBlank(venId)) {
      throw new BizMsgException("업체코드는 필수항목입니다.");
    }
    TenantRepItemDto result = tenantItemService.getEmartOfflineRepItemId(venId);
    return result;
  }
```

위와 같이 `Get` request를 Controller에 만들고, `@ResponseBody`를 내가 만든 POJO 클래스인 TenantRepItemDto로 리턴하는 API를 하나 만들었다.

그리고 View에서 ajax 콜을 다음과 같이 했다.

``` javascript
$.ajax({
    async       : true,
    url         : url,
    data        : (sendType === "form") ? param : JSON.stringify(param),
    traditional : true,
    type        : "GET",
    contentType : "application/x-www-form-urlencoded;",
    dataType    : "json",
    cache       : false,
    success     : function(result, status){
                        callbackFn(result, status);
                },
    error       : function(jqXhr, status, error){
                    if(errorCallbackFn){
                        errorCallbackFn(jqXhr, status, error);

                    }else{
                        alert("Ajax Call 오류가 발생했습니다.");
                    }
               }
});
```

데이터가 있을 경우에는 success 하지만, `tenantItemService.getEmartOfflineRepItemId(venId)` 값이 null일 경우 ajax 콜에서 계속 error로 떨어지는 이슈가 있었다.

### `@ResponseBody`가 있으면 `MessageConverter`가 실행됨

결과값이 null 이면 MessageConverter가 처리할 객체가 없어서 에러가 발생한다. [참고](https://moonsiri.tistory.com/25)

[이 포스팅](https://duooo-story.tistory.com/9) 에 보면, `@ResponseBody` 가 붙은 request를 DispatcherServlet이 어떻게 데이터를 리턴하는지 자세히 볼 수 있다.

본인은 F6을 누르면서 어떻게 동작하는지 확인했다. *(이게 더 쉬울수도..)*

handler가 어떤 messageConverters를 들고 있는지 확인할 수 있다.
![message_converter1](/assets/message_converter1.png)

그리고 해당 컨버터를 이용해서 response Data를 만든다.
![캡처](/assets/캡처.JPG)

만약 null인데도, 빈문자로 응답을 해서 정상 처리를 원할 경우 [이 포스팅](https://pmingdiary.tistory.com/22)에 적혀 있는데로 NullSerializer를 만들고, 커스텀 CustomObjectMapper 를 구현해주면 될것 같다.


### ajax에서 `dataType : json`은 empty가 될 수 없으므로 text로 바꿔준다.

[https://stackoverflow.com/questions/56139846/unexpected-end-of-json-input-on-a-void-springmvc-ajax-controller](https://stackoverflow.com/questions/56139846/unexpected-end-of-json-input-on-a-void-springmvc-ajax-controller)

### null이 될 수 없도록 Map을 활용한다.

필자는 Map 객체를 하나 만들고, 해당 객체를 리턴하는 식으로 바꿨다. (ResponseBody는 사용하려고)

``` java
@RequestMapping(value = "/getEmartOfflineRepItemInfo", method = RequestMethod.GET)
  public @ResponseBody Map<String, Object> getEmartOfflineRepItemInfo(@RequestParam("venId") String venId) {

    Map<String, Object> map = new HashMap<>();
    if(StringUtils.isBlank(venId)) {
      throw new BizMsgException("업체코드는 필수항목입니다.");
  }
  TenantRepItemDto result = tenantItemService.getEmartOfflineRepItemId(venId);
  map.put("repItemInfo", result);
    return map;
  }
```
