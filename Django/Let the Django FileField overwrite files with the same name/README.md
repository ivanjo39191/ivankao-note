# Let the Django FileField overwrite files with the same name

Created: February 1, 2022 5:22 PM
Tags: Django

```python
from app.storage import OverwriteStorage

class Thing(models.Model):
    image = models.ImageField(
        max_length=SOME_CONST, storage=OverwriteStorage(), upload_to=image_path
    )
```

```python
class OverwriteStorage(FileSystemStorage):
    def get_available_name(self, name, max_length=None):
        """
        Returns a filename that's free on the target storage system, and
        available for new content to be written to.

        Found at http://djangosnippets.org/snippets/976/

        This file storage solves overwrite on upload problem. Another
        proposed solution was to override the save method on the model
        like so (from https://code.djangoproject.com/ticket/11663):

        def save(self, *args, **kwargs):
            try:
                this = MyModelName.objects.get(id=self.id)
                if this.MyImageFieldName != self.MyImageFieldName:
                    this.MyImageFieldName.delete()
            except: pass
            super(MyModelName, self).save(*args, **kwargs)
        """
        # If the filename already exists, remove it as if it was a true file system
        if self.exists(name):
            os.remove(os.path.join(settings.MEDIA_ROOT, name))
        return name
```

[https://stackoverflow.com/questions/31666309/replacing-a-file-while-uploading-in-django](https://stackoverflow.com/questions/31666309/replacing-a-file-while-uploading-in-django)

[https://gist.github.com/fabiomontefuscolo/1584462](https://gist.github.com/fabiomontefuscolo/1584462)

[https://timonweb.com/django/imagefield-overwrite-file-if-file-with-the-same-name-exists/](https://timonweb.com/django/imagefield-overwrite-file-if-file-with-the-same-name-exists/)

[https://stackoverflow.com/questions/44389101/django-get-available-name-got-an-unexpected-keyword-argument-max-length](https://stackoverflow.com/questions/44389101/django-get-available-name-got-an-unexpected-keyword-argument-max-length)