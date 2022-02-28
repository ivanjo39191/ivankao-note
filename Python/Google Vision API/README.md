# Google Vision API

Created: February 1, 2022 5:22 PM
Tags: Python

**Vision API**

使用 Google 的預先訓練模型為圖片加上標籤，並將圖片分類成數百萬種預先定義的類別。另外還可偵測物件和臉孔、讀取印刷文字和手寫文字等。

1. Vision API Python 文檔
    
    [https://googleapis.dev/python/vision/latest/index.html](https://googleapis.dev/python/vision/latest/index.html)
    
2. 申請憑證
    
    GCP > API 和服務 > 憑證 > 建立憑證 > 服務帳戶
    
    ![DialogFlow%20dd21d/Untitled%203.png](DialogFlow%20dd21d/Untitled%203.png)
    
    建立後取得憑證的 json 檔案
    
    ```python
    {
      "type": "service_account",
      "project_id": "YOUR_PROJECT_ID",
      "private_key_id": "...",
      "private_key": "...",
      "client_email": "...",
      "client_id": "...",
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      "token_uri": "https://oauth2.googleapis.com/token",
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      "client_x509_cert_url": "..."
    }
    ```
    
    ```python
    import os, io
    from google.cloud import vision
    from django_q.tasks import async_task, result
    
    def vision_client(file_name):
        os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = '/opt/flysheetbot/src/df/ApponintmentScheduler.json'
        client = vision.ImageAnnotatorClient()
        # The name of the image file to annotate
        file_name = file_name
        pslog(file_name)
        # Loads the image into memory
        with io.open(file_name, 'rb') as image_file:
            content = image_file.read()
        image = vision.Image(content=content)
        # Performs label detection on the image file(output the possible label without score)
        response = client.label_detection(image=image)
        labels = response.label_annotations
        label_description = []
        for label in labels:
            label_description.append(label.description)
        # Performs web detection on the image file(I guess that the method just like google image search but capture the text from related site as output)
        response = client.web_detection(image=image,max_results=3)
        targets = response.web_detection.web_entities
        top_target= []
        for target in targets:
            top_target.append(target.description)
        # response = client.object_localization(image=image)
        # target2 = response.localized_object_annotations
    
        return label_description, top_target
    ```