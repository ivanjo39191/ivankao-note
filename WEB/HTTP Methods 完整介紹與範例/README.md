# HTTP Methods 完整介紹與範例

Created: May 30, 2022 10:23 PM  
Property: Ivan Kao  
Status: 已公開  
Tags: Web  

![Untitled](images/Untitled.png)

## 什麼是 HTTP ？

Hypertext Transfer Protocol (HTTP) 指實現 Web 客戶端與 Web 伺服器之間的溝通而設計，也可用於其他目的。

HTTP 用作客戶端和伺服器之間的請求-回應協議。

客戶端打開連接以發出請求，然後等待收到回應。HTTP 是一種無狀態協議，伺服器不會在兩個請求之間保留任何資料（狀態）。

舉例來說，從客戶端( Client ) 發送一個請求給伺服器，伺服器會根據請求而進行回應，回應內容包含請求的狀態碼與請求的內容。

## HTTP Methods 有哪些？

- **GET**           讀取資源
- **POST**        新增資源
- **PUT**           更新資源（完整更新）
- **HEAD**        檢查資源（沒有 body)
- **DELETE**    刪除資源
- **PATCH**      更換資源部分內容
- **OPTIONS**  回傳該資源所支援的所有 HTTP 請求方法
- **CONNECT** 將連線請求轉換至 TCP/IP 通道
- **TRACE**       用於 debug

## Get Method

**向指定的資源請求資料，類似於查詢操作。**

### GET 請求資料傳遞方式：

將參數以 Query String方式(name/value)，由URL帶至Server端

```python
GET /test/demo_get?name1=value1&name2=value2
```

### GET 請求的幾項特性：

- GET 請求可以被快取
- GET 請求保留在瀏覽器歷史記錄中
- GET 請求可以添加為書籤
- GET 請求有長度限制 — 長度限制根據瀏覽器、Server 的不同會有所不同。
- GET 請求僅用於請求數據（不修改）
- GET 請求的資料只允許 ASCII。
- GET 請求**可以**重新載入或按上一頁並不會有任何問題。
- 永遠不應該使用 GET 請求來處理敏感數據 — 傳遞的參數會在URL上顯示。

## **Post Method**

**將要處理的資料提交給指定的資源來創建或更新資料，類似於更新操作。**

### POST 請求資料傳遞方式：

將參數放至 Request 的 message body 中，因此不會在URL看到參數，適合用於隱密性較高的資料，例如 註冊、建立帳號、密碼等。

```python
POST /test HTTP/1.1
Host: foo.example
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

field1=value1&field2=value2
```

### POST 請求的幾項特性：

- POST 請求永遠不會被快取
- POST 請求不會保留在瀏覽器歷史記錄中
- POST 請求對資料長度沒有限制
- POST 請求傳遞的參數皆可以在封包(Request- line 和 Message-body)上看到。
- POST 請求沒有限制資料種類。
- POST 請求重新載入或按上一頁瀏覽器會出現將重新提交(re-submitted)資料的提示。
- POST 請求傳遞的參數**不會**被儲存在瀏覽器的歷史紀錄中。
- POST 請求**無法**加入瀏覽器書籤。

## GET 與 POST 比較

|  | GET | POST |
| --- | --- | --- |
| 返回上一頁或重新整理 | 可正常執行 | 資料會被重新送出 |
| 加入書籤 | 可加入書籤 | 不可加入書籤 |
| 快取 | 可被快取 | Not cached |
| 編碼類型 | application/x-www-form-urlencoded | application/x-www-form-urlencoded 
multipart/form-data(binary data) |
| 瀏覽器歷史紀錄 | 會保留在瀏覽器歷史記錄中 | 不會保留在瀏覽器歷史記錄中 |
| 資料長度限制 | 資料會添加到URL中，最大 URL 長度為 2048 個字元 | 無限制 |
| 資料類型 | 只允許 ASCII。 | 無限制，也允許二進制數據 |
| 安全性 | 比 POST 不安全，不應該使用 GET 請求來處理敏感數據  | 比 GET 安全一點，參數不會儲存在瀏覽器歷史記錄或伺服器紀錄中 |
| 可見度 | URL 中的每個人都可以看到資料 | 資料未顯示在 URL 中，但可以再封包中看到 |

## POST / PUT / PATCH 在 Restful 定義下的差異與使用時機

- 瀏覽全部資料：GET        + 資源名稱
- 瀏覽特定資料：GET        + 資源名稱 + :id
- 新增一筆資料：POST     + 資源名稱
- 修改特定資料：PUT        + 資源名稱 + :id
- 修改特定資料：PATCH   + 資源名稱 + :id
- 刪除特定資料：DELETE  + 資源名稱 + :id

## PUT Method

**將要處理的資料提交給指定的資源來創建或更新資料，類似於更新操作。**

POST 和 PUT 的區別在於 PUT 請求是冪等的（可使用相同參數重複執行）。 

多次調用同一個 PUT 請求總是會產生相同的結果。 

重複調用 POST 請求會產生多次創建相同資源的副作用。

```python
PUT /new.html HTTP/1.1
Host: example.com
Content-type: text/html
Content-length: 16

<p>New File</p>
```

## Head Method

**HEAD 幾乎與 GET 相同，但沒有回應 body。**

如果 GET /users 返回用戶列表，那麼 HEAD /users 將發出相同的請求，但不會返回用戶列表。

HEAD 請求可以在實際發出 GET 請求之前檢查 GET 請求將返回的內容，例如在下載大文件或回應 body 之前。

```python
HEAD /index.html
```

## DELETE Method

**DELETE 方法刪除指定的資源。**

```python
DELETE /file.html HTTP/1.1
Host: example.com
```

## PATCH Method

**PATCH 方法用於對資源應用部分修改。**

```python
PATCH /file.txt HTTP/1.1
Host: www.example.com
Content-Type: application/example
If-Match: "e0023aa4e"
Content-Length: 100

[description of changes]
```

## OPTIONS Method

**OPTIONS 方法回傳該資源所支援的所有 HTTP 請求方法。**

可以使用 curl 發出請求

```python
curl -X OPTIONS https://example.org -i
```

回應會包含一個包含 Allow 來顯示允許的方法：

```python
HTTP/1.1 204 No Content
Allow: OPTIONS, GET, HEAD, POST
Cache-Control: max-age=604800
Date: Thu, 13 Oct 2016 11:45:00 GMT
Server: EOS (lax004/2813)
```

## CONNECT Method

**CONNECT 方法用於啟動與請求資源的雙向通道( TCP / IP )。**

```python
CONNECT server.example.com:80 HTTP/1.1
Host: server.example.com:80
Proxy-Authorization: basic aGVsbG86d29ybGQ=
```

## **TRACE Method**

**TRACE 方法方法用於執行消息環回測試，以測試目標資源的路徑（用於 debug ）。**

```python
TRACE /index.html
```

## 參考資料

[https://www.w3schools.com/tags/ref_httpmethods.asp](https://www.w3schools.com/tags/ref_httpmethods.asp)

[https://developer.mozilla.org/en-US/docs/Web/HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)

[https://tw.alphacamp.co/blog/rest-restful-api](https://tw.alphacamp.co/blog/rest-restful-api)

[https://totoroliu.medium.com/http-post-和-get-差異-928829d29914](https://totoroliu.medium.com/http-post-%E5%92%8C-get-%E5%B7%AE%E7%95%B0-928829d29914)