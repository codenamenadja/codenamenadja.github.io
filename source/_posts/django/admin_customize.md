---
title: 장고에서 admin페이지 커스터마이징하기
p: django/admin_customize
date: 2019-06-14 13:41:47
tags: ['django']
---

1. order.admin수정해서 action에 행동 추가하기

```python
from django.utils import timezone
from django.http import HttpResponse
import datetime
import csv
def export_to_csv(modeladmin, request, queryset):
    # modeladmin.model - 선택된 객체의 모델 정보
    # request - request정보
    # queryset - 선택된 row들의 쿼리
    # 선택된 주문 정보를 CSV파일로 다운로드 하게 만드는 기능
    opts = modeladmin.model_meta

    response =HttpResponse(content_type='text/csv')
    current_time = timezone.now().strftime('%Y-%m-%d %H:%M:%S')

    response['Content-Disposition'] = f'attachment;filename={opts.verbose_name}-{current_time}.csv'
    writer = csv.writer(response)

    # 필드 값 작성
    fields = [field for field in opts.get_fields() if (not field.many_to_many) and (not field.one_to_many)]
    writer.writerow([field.verbose_name for field in fields])
    
    # 데이터 작성
    for obj in queryset:
        # 선택된 요소들에 대한 내용을 출력
        ordered_items = getattr(obj, 'items').all()
        values = [getattr(obj, field.name).strftime('%Y-%m-%d') if field.datetime else getattr(obj, field.name) for field in fields]
        for item in ordered_items:
            order_detail = [item.product.id, item.price, item.quantity]
            writer.writerow(values+order_detail)
export_to_csv.short_description = 'Order Export to CSV' # 여

class OrderOption(admin.ModelAdmin):
    """
    ...
    """
    actions = [export_to_csv]
```

2. Page 추가하기

```python
from django.utils.safestring import mark_safe

def order_detail(obj):
    href = reverse('order:admin_detail')
    return mark_safe(f'<a href="{href}">to Detail</a>')

order_detail.short_description = 'details'

def order_pdf(obj):
    return mark_safe('PDF')

order_pdf.short_description = 'to_pdf'

OrderOption.list_display += [order_pdf, order_detail]
```


```python
# order.views.py
from django.shorcuts import get_object_or_404
from django.contrib.admin.views.decorators import staff_member_required

@staff_member_required
def admin_order_detail(req, order_id):
    order = get_object_or_404(Order, id=order_id)
    return render(req, 'order/admin/order_detail.html', {'order':order})

# order.urls.py
app_name = 'order'
urlpatterns += [
    path('admin/detail/<int:order_id>', admin_order_detail, name='order_detail')
]
```


```html
{%extends 'admin/base_site.html'%}

{%block breadcrumbs%}
<div class='breadcrumbs'>
    <a href='{url "admin:index"}'></a>
    <a href='{url "admin:order_order_changelist"}'>Orders</a>
    <a href='{url "admin:order_order_change" order.id}'>Order {{order.id}}</a>
    Detail
 </div>
{%endblock%}

{%block content%}
// order객체 가지고 표시
{%endblock%}
```

3. weasyprint로 PDF출력하기

pip install weasyprint
```python
from django.template.loader import render_to_string
from django.http import HttpResponse
import weasyprint

@staff_member_required
def admin_order_pdf(req, order_id):
    order = get_object_or_404(Order, id=order_id)
    html = render_to_string('order/admin/pdf.html', {'order':order})
    response = HttpResponse(content_type='application/pdf')
    response['Content-Disposition'] = f'filename=invoice_{order.id}.pdf'
    weasyprint.HTML(string=html).write_pdf(response)
    return response
```

- 다운로드 Disposition = attachment
- 웹 뷰어로 보기 Disposition = invoice
