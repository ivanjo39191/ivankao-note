# DRF viewsets retrieve filter

Created: February 1, 2022 5:22 PM
Tags: Django

DRF 篩選特定欄位

pk 為傳入特定值

使用 Place 的 json_d 進行 filter

```jsx
class PlaceViewSet(viewsets.ModelViewSet):
    queryset = Place.objects.all()
    serializer_class = PlaceSerializer
    permission_classes = (AllowAny, )

    def retrieve(self, request, pk=None):
        queryset = Place.objects.all()
        place = Place.objects.get(json_d=pk)
        serializer = PlaceSerializer(place)
        return Response(serializer.data)
```

測試網址如下:

[http://ttime.ivankaoblog.com:8000/api/travel/places/C1_315081800H_000071/](http://ttime.ivankaoblog.com:8000/api/travel/places/C1_315081800H_000071/)