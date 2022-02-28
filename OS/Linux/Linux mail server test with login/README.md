# Linux mail server test with login

Created: February 1, 2022 5:22 PM
Tags: OS

```python
telnet mailserver.com 25
EHLO mailserver.com
# 回應
250-mailserver.com
250-PIPELINING
250-SIZE 50000000
250-VRFY
250-ETRN
250-AUTH PLAIN LOGIN
250-AUTH=PLAIN LOGIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
# 登入 ,如果是 base64 須將密碼 encode
AUTH LOGIN
# 輸入帳號
334 TESTcm5hbW**
aT5mb3EtSTRpb2**
# 輸入密碼
334 TESzc3tvcm**
T211bEliST**
# 失敗訊息
535 5.7.8 Error: authentication failed: authentication failure
```

Referfence

[https://chengfulin.github.io/posts/筆記-如何用Telnet測試SMTP-AUTH/](https://chengfulin.github.io/posts/%E7%AD%86%E8%A8%98-%E5%A6%82%E4%BD%95%E7%94%A8Telnet%E6%B8%AC%E8%A9%A6SMTP-AUTH/)