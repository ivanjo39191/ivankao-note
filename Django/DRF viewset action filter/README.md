# DRF viewset action filter

Created: February 1, 2022 5:22 PM
Tags: Django

1. 使用 Django-filter 進行篩選
    
    ```python
    import django_filters
    
    class NameFilter(django_filters.FilterSet):
        def __init__(self, *args, name=None, **kwargs):
            super().__init__(*args, **kwargs)
        name = django_filters.Filter(field_name='name', lookup_expr='icontains')
        class Meta:
            fields = ['name']
    
    class RegionFilter(django_filters.FilterSet):
        def __init__(self, *args, region=None, **kwargs):
            super().__init__(*args, **kwargs)
        region = django_filters.Filter(field_name='region', lookup_expr='icontains')
        class Meta:
            fields = ['region']
    
    class PlaceFilter(NameFilter, RegionFilter):
        class Meta:
            models = Place
            fields = ['name', 'region']
    ```
    
2. ViewSet action 使用篩選器
    1. lookup_field 更改預設的單實例查找欄位(預設為pk)
    2. get_serializer_class 根據 action 行為改變使用的 serializer
    3. filter_queryset 根據 action 行為改變使用的 filter 
    4. @action(detail=False) 為 list_route，queryset 調用改寫過的 filter_queryset 
    
    ```python
    from rest_framework import viewsets, status, filters
    from rest_framework.response import Response
    from rest_framework.decorators import action
    
    class PlaceViewSet(viewsets.ModelViewSet):
        queryset = Place.objects.all()
        permission_classes = (IsAuthenticatedOrReadOnly, )
        filterset_class  = PlaceFilter
        lookup_field='json_d'
        def get_serializer_class(self):
            if hasattr(self, 'action') and self.action == 'list':
                return serializers.PlaceSerializer
            if hasattr(self, 'action') and self.action == 'retrieve':
                return serializers.PlacePositionSerializer
    
        def filter_queryset(self, queryset):
            if self.action == 'list':
                self.filterset_class = PlaceFilter
            elif self.action == 'position':
                self.filterset_class = RegionFilter
            return super().filter_queryset(queryset)
    
        @action(detail=False)
        def position(self, request):
            queryset = self.filter_queryset(self.get_queryset())
            serializer = serializers.PlacePositionSerializer(queryset, many=True)
            return Response(serializer.data)
    ```