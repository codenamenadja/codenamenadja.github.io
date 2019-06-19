---
title: django에서 페이지네이션 구현하기
p: django/paginator
date: 2019-05-31 13:36:20
tags: ['django', 'pagination']
---

1. Paginator 사용하기
```python
from django.core.paginator import Paginator

pager = paginator(Post.objects.all(), 10, allow_empty_first_page=True)
# Post.objects.all().filter() 로 줄이고 할 수 있다.

curpage = 1
>>> Page = pager.page(curpage) # Objects 로 객체들

>>> Page.object_list
<QuerySet>

>>> Page.has_next()
True

>>> Page.has_previous()
False

>>> pager.count
3 #전체 객체 수

>>> pager.num_pages
2 # 페이지 수

```


```py
# list view 수정

class PostList(ListView):
    model = Post
    template_name = ''
    # 페이지 기능 활성화
    paginated_by = 1
    page_kwarg = 'p'
    # /list/?p=2

    # urls.py /list/<int:p>/
    # /list/2/
```
```html
# template layout
{% if is_paginated %}
    ul.pagination justify-content-center pagination-sm
        {%if page_obj.has_previous%}
            li.page-item
                a.page-link[href='?page={{page_obj.previous_page_number}}']
                    Previous Page
                /a
            /li             
        {%endif%}
        
        {%for page_num in paginator.page_range%}
            li.page-item
                a.page-link[href='?page={{page_num}}']
                    {{page_num}}
                /a
            /li
        {%endfor%}


        {%if page_obj.has_next%}
            li.page-item
                a.page-link[href='?page={{page_obj.next_page_number}}']
                    Next Page
                /a
            /li             
        {%endif%}
   /ul
{% endif%}
<--해당 뷰가 class에 pagenated_by 속성이 지정되었을 경우 페이지 네이션 기능을 초기화-->
```

- 함수형 뷰로 페이지 구현하기
```python
# querystring target_pag, pagenated_by 사용 유저가 자유롭게 선택할 수 있고, 값이 없을 경우에는 기본값을 할당?

from django.core.paginator import Paginator

def postList(req):
    
    target_page = int(req.GET.get('target_page', 1)) # 아래 로직을 없애줌
    pagenated_by = int(req.GET.get('pagenated_by', 10)) # 부재시 기본값 설정
    """
    if not target_page:
        target_page = 1
    if not pagenated_by:
        pagenated_by = 10
    """
    object_list = Post.objects.all()
    paginator = Paginator(object_list, pagenated_by, allow_empty_first_page=True)
    page_obj = paginator.page(target_page)
    if not page_obj.object_list: # 10개를 요청하고 2페이지를 요청했을때,
       page_obj = paginator.page(1)

    return render(req, 'post/list.html', {'paginator':paginator, 'page_obj':page_obj, 'is_paginated':True})
```

```python
# urls.py
# 한줄로 하는 방식
[re_path(r'^(?P<page>[0-9]+)*/{0,1}$', postList, name='post_list_fbv'),

# 두 줄로 하는 방식
path('<int:p>/', Post_List.as_view(), name='post_list_cbv'),
path('', Post_List.as_view(), name='post_list_cbv') 
]

```
