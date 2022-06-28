---
title:  "[Django] GenericView 동작방법"
date: 2022-02-08
excerpt: ""
tags: [til, python]
categories: [til/python]
---

# Python Class-based View
`GenericView` 를 쓰고 있는 class 에 `delete` 메소드가 있었는데, delete 요청을 보내면 어떻게 해당 함수를 매핑하는지가 궁금했다. 왜냐하면 [Viewsets - Django REST framework](https://www.django-rest-framework.org/api-guide/viewsets/#genericviewset) 에서는 아래 항목을 지원한다고 적혀있기 떄문이다.
```
class UserViewSet(viewsets.ViewSet):
    """
    Example empty viewset demonstrating the standard
    actions that will be handled by a router class.

    If you're using format suffixes, make sure to also include
    the `format=None` keyword argument for each action.
    """

    def list(self, request):
        pass

    def create(self, request):
        pass

    def retrieve(self, request, pk=None):
        pass

    def update(self, request, pk=None):
        pass

    def partial_update(self, request, pk=None):
        pass

    def destroy(self, request, pk=None):
        pass

```

어떻게 `delete` 함수가 매핑되는 것일까? 먼저 `urls.py` 에서 해당 ViewSet을 router 에 의해 url이 매핑되는 것을 확인했다. 그리고 `router.urls` 로 url 매핑된 것을 찾았다.
```
<URLPattern '^abc/?$' [name='abc-list']>,
 <URLPattern '^abc/confirm/?$' [name='abc-confirm']>,
 <URLPattern '^abc/detail/?$' [name='abc-detail-info']>,
 <URLPattern '^abc/recover/?$' [name='abc-recover']>,
 <URLPattern '^abc/verify/?$' [name='abc-verify']>,
 <URLPattern '^abc/(?P<pk>[^/.]+)/?$' [name='abc-detail']>,
```

하지만 이렇게 보면 custom한 엔드포인트와, —list, —detail 엔드포인트만 보여주지 어떤 method 에 매핑되는지는 적혀있지 않다.
따라서 ViewSet 인스턴스 자체를 가져와서 확인해보기로 했다.

![](56C4EC9C-6531-4046-930D-48B03F0FA923.png)

에러가 뜬다. `as_view()` 는 {‘get’: ‘;list’} 처럼 어떤 메서드로 요청되었을 때 어떤 함수를 실행하라 라는 파라미터를 주고 view를 만드는 함수이기 떄문이다.

그냥 ViewSet 자체를 보면 어떻게 나오는지 실행해봤다.

![](22C3CD92-24DA-428D-A6A9-F9D68FB68C59.png)

유의미한 것이 보였다! ViewSet자체에서 delete 함수를 가지고 있는 것이다. 그러면 이게 어떻게 가능한걸까? `GenericViewSet` 이 상속하고 있는 `View` 클래스를 보면 dispatch 함수가 있다.

``` python
def dispatch(self, request, *args, **kwargs):
    # Try to dispatch to the right method; if a method doesn't exist,
    # defer to the error handler. Also defer to the error handler if the
    # request method isn't on the approved list.
    if request.method.lower() in self.http_method_names:
        handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
    else:
        handler = self.http_method_not_allowed
    return handler(request, *args, **kwargs)

```

여기서 `delete` 요청이 오면 delete 함수를 찾아서 보내주고 있는 것이다.  그래서 def put, def patch 등을 이용해도 동일한 효과를 내주고 있는 것을 확인했다. `getattr` 의 의미만 알아도 조금 더 일찍 알 수 있지 않았을까 싶다.
