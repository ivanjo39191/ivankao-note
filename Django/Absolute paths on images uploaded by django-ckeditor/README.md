# Absolute paths on images uploaded by django-ckeditor

Created: February 1, 2022 5:22 PM
Tags: Django

I would use a custom serializer to fix that:

```
from rest_framework import serializers

def relative_to_absolute(url):
    return 'http://127.0.0.1:8000' + url

class FileFieldSerializer(serializers.Field):
    def to_representation(self, value):
        url = value.url
        if url and url.startswith('/'):
            url = relative_to_absolute(url)
        return url

```

When filefield.url contains a relative url, **relative_to_absolute()** is called to prepend the domain.

Here I just used a constant string; you can either save it in your settings, or, if Django Site framework is installed, retrieve it as follows:

```
from django.contrib.sites.models import Site

domain=Site.objects.get_current().domain

```

Sample usage of the custom serializer:

```
class Picture(BaseModel):
    ...
    image = models.ImageField(_('Image'), null=True, blank=True)
    ...

class PictureSerializer(serializers.ModelSerializer):
    image = FileFieldSerializer()
    class Meta:
        model = Picture
        fields = '__all__'

```

## Variation for RichTextUploadingField

If, on the other hand, you're using RichTextUploadingField by CKEditor, your field is, basically, a TextField where an HTML fragment is saved upon images upload.

In this HTML fragment, CKEditor will reference the uploaded images with a relative path, for very good reasons:

- your site will still work if the domain is changed
- the development instance will work in localhost
- after all, we're using Django, not WordPress ;)

So, I wouldn't touch it, and fix the path at runtime in a custom serializer instead:

```
SEARCH_PATTERN = 'href=\\"/media/ckeditor/'
SITE_DOMAIN = "http://127.0.0.1:8000"
REPLACE_WITH = 'href=\\"%s/media/ckeditor/' % SITE_DOMAIN

class FixAbsolutePathSerializer(serializers.Field):

    def to_representation(self, value):
        text = value.replace(SEARCH_PATTERN, REPLACE_WITH)
        return text

```

Alternatively, domain can be saved in settings:

```
from django.conf import settings

REPLACE_WITH = 'href=\\"%s/media/ckeditor/' % settings.SITE_DOMAIN

```

or retrieved from Django Site framework as follows:

```
from django.contrib.sites.models import Site
REPLACE_WITH = 'href=\\"{scheme}{domain}/media/ckeditor/'.format(
    scheme="http://",
    domain=Site.objects.get_current().domain
)

```

**You might need to adjust `SEARCH_PATTERN` according to your CKEditor configuration; the more specific, the better.**

Sample usage:

```
class Picture(BaseModel):
    ...
    body = RichTextUploadingField(blank=True,null=True)
    ...

class PictureSerializer(serializers.ModelSerializer):
    body = FixAbsolutePathSerializer()
    class Meta:
        model = Picture
        fields = '__all__'
```

[https://stackoverflow.com/questions/61325006/absolute-paths-on-images-uploaded-by-django-ckeditor](https://stackoverflow.com/questions/61325006/absolute-paths-on-images-uploaded-by-django-ckeditor)