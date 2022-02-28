# Timeout when reading response headers from daemonrocess

Created: February 1, 2022 5:22 PM
Tags: OS

Timeout when reading response headers from daemonrocess 'ivankaobot'

```python
[Fri Apr 30 13:02:46.412937 2021] [wsgi:error] [pid 2385213:tid 14054647128243 [client 101.10.63.75:64895] Timeout when reading response headers from daemonrocess 'ivankaobot': /path/wsgi.py, referer: https://example.com/dialogflow/
```

[https://stackoverflow.com/questions/40413171/django-webfaction-timeout-when-reading-response-headers-from-daemon-process](https://stackoverflow.com/questions/40413171/django-webfaction-timeout-when-reading-response-headers-from-daemon-process)

```python
WSGIApplicationGroup %{GLOBAL}
```

```python
<IfModule !wsgi_module>
    LoadFile /usr/local/lib/libpython3.8.so.1.0
    LoadModule wsgi_module /usr/local/apache2.4/modules/mod_wsgi.so
</IfModule>
<Directory "/opt/flysheetbot/src/">
    <Files wsgi.py>
        Require all granted
    </Files>
</Directory>

...

WSGIDaemonProcess bot user=username group=usergroup socket-user=daemon
WSGIProcessGroup bot
WSGIApplicationGroup %{GLOBAL}

WSGIScriptAlias / /path/wsgi.py
```