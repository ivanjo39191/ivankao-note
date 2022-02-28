# Django get_readonly_fields

Created: February 1, 2022 5:22 PM
Property: ivan kao
Tags: Django

分為創建時唯獨、修改時唯獨

```python
def get_readonly_fields(self, request, obj=None):
        create_readonly_fields = (
            'created',
            'modified',
        )
        readonly_fields = (
            'primary_id',
            'created',
            'modified',
        )
        if obj:  # editing an existing ob
```