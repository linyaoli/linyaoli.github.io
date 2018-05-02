# nginx's add_header is not working

Try this:

```
set $content_disposition $upstream_http_content_disposition;
...
add_header Content-Disposition $content_disposition;
```
