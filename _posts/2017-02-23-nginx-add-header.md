---
layout: post
title: nginx's add_header is not working
tags: programming
---

Just in case anyone turns into a clusterfuck that:
NO, stackoverflow doesn't help!(just like me)
try this:
~~~
set $content_disposition $upstream_http_content_disposition;
...
add_header Content-Disposition $content_disposition;
~~~
