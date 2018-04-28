# What happens when you type a URL in the browser

A URL may contain request to HTML, image file or any other type.


1. If content of the typed URL is in cache and fresh, then display the content.
2. Else find IP address for the domain so that a TCP connection can be setup. Browser does a DNS lookup.
3. Browser needs to know IP address for a url, so that it can setup a TCP connection.  This is why browser needs DNS service.  Browser first looks for URL-IP mapping browser cache, then in OS cache. If all caches are empty, then it makes a recursive query to the local DNS server.   The local DNS server provides the IP address.
4. Browser sets up a TCP connection using three way handshake.
5. Browser sends a HTTP request.
6. Server has a web server like Apache, IIS running that handles incoming HTTP request and sends a HTTP response.
7. Browser receives the HTTP response and renders the content.
