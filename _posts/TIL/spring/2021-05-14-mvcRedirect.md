---
title:  "[Web] Spring Redirects"
date: 2021-04-10
excerpt: ""
tags: [til, spring]
categories: [til/spring]
---

## 개요

A 페이지에서 B페이지로 `redirection`하는 목적은 여러가지가 있을 수 있습니다. 단순히 form data를 POSTing 하거나, 흐름을 다른 controller로 위임하고자 할 때 url을 리다이렉팅합니다.


A quick side note here is that the typical Post/Redirect/Get pattern doesn't adequately address double submission issues – problems such as refreshing the page before the initial submission has completed may still result in a double submission.

{: .notice--info}

## 1. RedirectView 사용

``` java
@Controller
@RequestMapping("/")
public class RedirectController {

    @GetMapping("/redirectWithRedirectView")
    public RedirectView redirectWithUsingRedirectView(
      RedirectAttributes attributes) {
        attributes.addFlashAttribute("flashAttribute", "redirectWithRedirectView");
        attributes.addAttribute("attribute", "redirectWithRedirectView");
        return new RedirectView("redirectedUrl");
    }
}
```

`org.springframework.web.servlet.view` 패키지에 있는 `RedirectView` 를 사용하면 스프링 서블릿이 자동으로 `HttpServletResponse.sendRedirect()`를 실행해서 리다이렉팅을 해줍니다.

### RedirectAttributes

`RedirectAttributes` 를 통해서 Target url로 파라미터를 넘길 수 있습니다. 또한, 넘겨진 파라미터를 redirected 된 페이지에서 정보를 노출시키지 않을수도 있습니다.

``` java
attributes.addFlashAttribute("flashAttribute", "redirectWithRedirectView");
attributes.addAttribute("attribute", "redirectWithRedirectView");
```

주의해야할점은, `addAttribute`로 값을 전달할 때는 `Primitive Type`만 가능하다는 점입니다. 즉, `String`, `Integer` 과 같은 값만 전달 가능하며 사용자가 직접 정의한 Dto를 전달할 수 없습니다. `Object`를 전달하고자 할 때는 `FlashAttribute를 사용하면 됩니다.`

### FlashAttribute

`FlashAttribute`는 문자열로 구성된 Parameter로 데이터가 넘어가는것이 아닌, Session을 이용해 데이터를 전달하는데, 이 때 넘겨받은 핸들러에서만 사용 가능하고, 넘겨받은 핸들러가 종료될때 Session에서 데이터가 삭제되기 때문에 값을 유지할 수 없다는 점이 있습니다.

``` java
attributes.addFlashAttribute("flashAttribute", test);
```

이렇게 값을 전달해서 요청을 받게된 타깃 컨트롤러에서는 `Model`객체에 자동으로 등록됩니다.

``` java
@RequestMapping(value = { "/target" }, method = RequestMethod.GET)
public String itemNew(ModelMap model) {

    if(model.get("flashAttribute") != null) {
        System.out.print(model.get("flashAttribute"));
    }
}
```

[참고] (https://galid1.tistory.com/563)


## 2. `redirect` prefix를 사용

위에서 언급한 `RedirectView`는 최적의 방법은 아닙니다.
- Spring API를 직접 우리 코드에 넣어야 한다.
- `RedirectView`가 무조건 redirecting 된다는 사전지식을 가지고 있어야 한다.

보다 좋은 방법은 prefix `redirect`를 사용하는 것입니다. MVC 패턴으로 리턴하는 View 이름에 `redirect`를 붙이기만 하면 끝납니다.

``` java
@RequestMapping(value = "/tenant/goItemByGate", method = RequestMethod.POST)
public ModelAndView itemGate(TenantSearchInfo tenantSearchInfo, ModelMap model) {
    model.put("oreItemInfo", tenantSearchInfo);
    return new ModelAndView("redirect:/cp/item/item/itemNew.ssg?itemRegDiv=40");
}
```

리턴되는 url에 `redirect`가 붙게된다면 `UrlBasedViewResolver` 가 자동으로 리다이렉션을 한다는 지시를 인지하고 redirect 뒤에 있는 url로 붙어있는 view를 찾게됩니다.

## 참고자료

https://www.baeldung.com/spring-redirect-and-forward
https://galid1.tistory.com/563
