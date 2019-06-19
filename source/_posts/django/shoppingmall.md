---
title: post
p: django/shoppingmall
date: 2019-06-14 10:38:58
tags:
---

1. IAMORT - 결제 정보 모델 로컬 데이터베이스 저장 이전에 pre_save Validation

```python
def order_payment_validation(sender, instance, created, *args, **kwargs):
    if instance.transaction_id: 
        import_transaction = OrderTransaction.objects.get_transaction(merchant_order_id = instance.merchant_order_id)

        merchant_order_id = import_transaction['merchant_order_id']
        imp_id = import_transaction['imp_id']
        amount = import_transaction['amount']
 
        # post_save
        """
        local_transaction = OrderTransaction.objects.filter(mechant_order_id = merchant_order_ID, transaction_id = imp_id, amount = amount).exists()

        if not import_transaction or not local_transaction:
            raise ValueError('비정상 거래')
        """
        # pre_save
        match_order_id = (instance.merchant_order_id != merchant_order_id)
        match_tran_id = (instance.transaction_id != imp_id)
        match_amount = (instance.amount != amount)
        is_not_valid = any([match_order_id, match_tran_id, match_amount])
        )]

        if is_not_valid:
            if instance.order.paid == False:
                data = cancel_transaction(instance.transaction_id)
                raise ValueError('비정상 거래 결제 취소')
            raise ValueError("비정상 거래 주문 실패")

from django.db.models.sigmals import post_save, pre_save
#post_save (.save() and do this)
"""
post_save.connect(order_payment_validation, sender=OrderTransaction)
"""
pre_save.connect(order_payment_validation, sender=OrderTransaction)
```

1. IAMPORT - 결제정보 로컬 저장 DB모델과 Object매니저

```python
class OrderTransactionManager(models.Manager):
    def create_new(self, order, amount, success=None, transaction_status=None):
        if not order:
            raise ValueError("주문이 존재 하지 않습니다.")

        temp_uuid = uuid.uuid1()
        temp_order_id = (str(temp_uuid)+str(order.email)).encode('utf-8')
        hashed_order_id = hashlib.sha1(temp_order_id).hexdigest()[:10]
        merchant_order_id = str(hashed_order_id)
        payment_prepare(merchant_order_id, amount)

        transaction = self.model(
            order=order,
            merchant_order_id=merchant_order_id,
            amount=amount
        )

        if success is not None:
            transaction.success = success
            transaction.transaction_status = transaction_status

        try:
            transaction.save()
        except Exception as e:
            print("save error", e)

        return transaction.merchant_order_id

    def get_transaction(self, merchant_order_id):
        result = find_transaction(merchant_order_id)
        if result['status'] == 'paid':
            return result
        else:
            return None

class OrderTransaction(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name='transaction')
    merchant_order_id = models.CharField(max_length=20, blank=True, null=True)
    transaction_id = models.CharField(max_length=120, blank=True, null=True)
    amount = models.IntegerField(default=0)
    transaction_status = models.CharField(max_length=20, blank=True, null=True)
    type = models.CharField(max_length=100, blank=True, null=True)
    created = models.DateTimeField(auto_now_add=True)

    objects = OrderTransactionManager()

    def __str__(self):
        return str(self.order.id) + "'s Transaction"

    class Meta:
        ordering = ['-created']
```
