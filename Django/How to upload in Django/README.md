# How to upload in Django

Created: February 1, 2022 5:22 PM
Tags: Django, Python

1. models.py
    
    ```python
    class Vision(models.Model):
        file = models.FileField("Vision", null=True, blank=True, 
        upload_to=helpers.upload_handle)
    ```
    
2. forms.py
    
    ```python
    class VisionForm(forms.Form):
        file = forms.FileField(
            label='Select a file',
            help_text='max. 42 megabytes'
        )
    ```
    
3. views.py
    
    ```python
    file_input = request.FILES['file']
    form = VisionForm(request.POST, request.FILES)
    if form.is_valid():
        newdoc = Vision(file = request.FILES['file'])
        newdoc.save()
    ```
    

[https://stackoverflow.com/questions/5871730/how-to-upload-a-file-in-django](https://stackoverflow.com/questions/5871730/how-to-upload-a-file-in-django)