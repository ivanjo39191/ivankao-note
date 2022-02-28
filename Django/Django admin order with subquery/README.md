# Django admin order with subquery

Created: February 1, 2022 5:22 PM
Tags: Django

Django Admin 計算另外一張表來進行排序

範例、

管理端的 Month 要計算 Statisics 的 type 為 ‘month’ 時的數量與流量並可進行排序。

```python
# models.py

class Statisics(models.Model):

    type = models.CharField('類型', max_length=11)
    tid = models.CharField('ID', max_length=11)
    tname = models.TextField('名稱')
    amount = models.IntegerField('數量', max_length=20)
    bytes = models.FloatField('流量', max_length=20)

    class Meta:
        verbose_name = '紀錄'
        verbose_name_plural = '紀錄'

    def __str__(self):
        return "%s" % (self.id)

class Month(models.Model):
    month = models.CharField('月份', max_length=10, unique=True)

    class Meta:
        verbose_name = '月份流量'
        verbose_name_plural = '月份流量'

    def __str__(self):
        return "%s" % (self.month)
```

```python
# admin.py

@admin.register(Month)
class MonthAdmin(admin.ModelAdmin):
    
    search_fields = ('month',)
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def get_queryset(self, request):
        queryset = super().get_queryset(request)
        
        sub = Statisics.objects.filter(Q(type='month') & Q(tid=OuterRef('id')))
        queryset = queryset.annotate(
            _bytes=Subquery(sub.values('bytes')), 
            _count=Subquery(sub.values('amount'))
        )
        return queryset
    # 定義 count，obj 返回上方 annotate 的 _count
    def count(self, obj):    
        if obj._count:
            return obj._count
        else:
            return 0

    # 定義 bytes，obj 返回上方 annotate 的 _bytes
    def bytes(self, obj):
        if obj._bytes:
            return obj._bytes
        else:
            return 0

    log_detail_count.short_description = '數量'
    log_detail_bytes.short_description = '流量'
    list_display = ['month','count','bytes']
    # 定義 admin_order_field 
    log_detail_count.admin_order_field = '_count'
    log_detail_bytes.admin_order_field = '_bytes'

```