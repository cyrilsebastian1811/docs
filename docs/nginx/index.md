# __:simple-nginx:{.lg .top} Nginx__

What is Nginx?

1. It's can be used as a Web Server to serve web content
2. It's can used as a Reverse Proxy to perform the following:
    1. Load Balancing between several backend resources
    2. Backend routing based on path(ex: /books, /clothes) or host(ex: one.example.com, two.example.com)
    3. Caching: used to serve relatively static content to reduce latency
    4. API Gateway: rate limiting, version based routing

Nginx operates primarily at Layer 4 (Transport Layer) or Layer 7 (Application Layer) of the OSI model or both.

- Layer 4: It behaves like a TCP/UDP proxy. And has access to:
    - Source IP and Port
    - Destination IP and Port
    - Simple packer inspection (SYN/TLS hello)
- Layer 7: Performs traffic managment by inspecting application layer data:
    - header
    - URL's
    - cookies
    - HTTP
    - HTTPs

### Transport Layer Security (TLS)

It's a way to establish end-to-end ancryption between a client and a server. These are the steps involved in the process:

- Symmetric Encryption: data is encrypted using an encryption key to produce cipher data
- Asymmetric Encryption: The encrypted key is encrypted using public key to produce cipher key. The cipher data and cipher key is sent to the server. and the revers is performed to decrypt the data.
- Certificate verification: The public key of server is signed by a trusted CA and subsequently produced my the server to clients. The client can ensure trust and authenticity of server by delegating verification to CA.


How does Nginx handle TLS termination:

1. When Nginx has TLS(e.g. HTTPS), and backend does not (HTTP). Then Nginx terminates TLS and decrypts and sends unencrypted data to backend.
2. When Nginx has TLS(e.g. HTTPS), and backend has (HTTPS). Then Nginx terminates TLS and decrypts, optionally mutates the data, and then re-encrypts before sending to backend.
3. TLS Passthrough: Nginx can just pass HTTPS request along


### Key components of Nginx

=== "Context"

    Contexts in NGINX are the hierarchical blocks in which you place related configuration directives. Different contexts serve different purposes, and ==certain directives are only valid within specific contexts==.

    Common Nginx Contexts:

    1. `main`: highest-level context, which applies to the entire NGINX instance. Exists at the top of the nginx.conf file, not enclosed in `{}`. Only ___centain directives are allowed in this context___. [Allowed directives](https://nginx.org/en/docs/ngx_core_module.html)

    2. `events`: How NGINX handles connections, like maximum connections and multi-threading configurations.

    3. `http`: Settings related to handling HTTP requests.

    4. `server`: Specifies configuration for a specific virtual server. Each virtual server listens for a specific domain or IP address. ___Server context is nested inside the http context___.

    5. `location`: Defines how NGINX handles requests for specific URIs or file paths. Can be nested inside the server context to define how certain paths (e.g., `/images` or `/api`) should be handled.

    6. `upstream`: Defines a group of backend servers that NGINX can proxy requests to. It's useful for load balancing.

    7. `if`: A conditional block that can be used to define rules for handling specific requests.


=== "Directives"

    Directives are the specific instructions that NGINX references to handles requests, manages processes, logs data, etc. Directives take parameters and are ==used inside contexts== to apply configurations.

    __Types of Directives:__

    1. Simple Directives: These consist of a name and one or more parameters. The line ends with a `;`.
    2. Block Directives: These contain a set of additional directives enclosed within curly braces `{}`. Block directives define a scope and can include both simple and block directives inside.


    __Common directives:__

    ??? "[`user`](https://nginx.org/en/docs/ngx_core_module.html#user)"

        Specifies the user and group under which NGINX worker processes run.

        ```{ .lua }
        Syntax:	    user user [group];
        Default:	user nobody nobody;
        Context:	main

        user www-data www-data;
        ```

    ??? "[`include`](https://nginx.org/en/docs/ngx_core_module.html#include)"

        Includes external configuration files into the current NGINX configuration.

        ```{ .lua }
        Syntax:	    include file | mask;
        Default:	—
        Context:	main, http, server, location

        include /etc/nginx/mime.types;
        include /etc/nginx/conf.d/*.conf;
        ```

    ??? "[`error_page`](https://nginx.org/en/docs/http/ngx_http_core_module.html#error_page)"

        ```{ .lua }
        Syntax:	    error_page code ... [=[response]] uri;
        Default:	—
        Context:	http, server, location, if in location

        error_page 404             /404.html;
        error_page 500 502 503 504 /50x.html;
        ```

    ??? "[`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log)"

        is used to specify the file where NGINX writes error messages and the logging level to control the verbosity of the logs.

        ```{ .lua }
        Syntax:	    error_log file [level];
        Default:	error_log logs/error.log error;
        Context:	main, http, mail, stream, server, location

        error_log /var/log/nginx/error.log;
        error_log /var/log/nginx/error.log info;
        ```


    ??? "[`access_log`](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log)"

        ```{ .lua }
        Syntax:	    access_log path [format [buffer=size]] [gzip[=level]] [flush=time] [if=condition];
        Default:	access_log logs/access.log combined;
        Context:	http, server, location, if in location
        ```

        Configures the path to the log file and the format of log entries.

        Example:
        ```nginx
        access_log /var/log/nginx/access.log combined;
        access_log /var/log/nginx/special.log special_format;
        ```

    ??? "[`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server)"

        Defines the configuration for a virtual server within the `http` context.

        ```{ .lua }
        Syntax:	    server { ... }
        Default:	—
        Context:	http

        server {
            listen 80;
            server_name example.com;
            root /var/www/html;
            location / {
                try_files $uri $uri/ =404;
            }
        }
        ```

    ??? "[`listen`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#listen)"

        ```{ .lua }
        Syntax:	    listen address[:port] [parameters];
        Default:	listen *:80;
        Context:	upstream

        listen 127.0.0.1:8080;
        listen [::1]:8080 ssl http2;
        listen 443 ssl;
        ```

        - **`address[:port]`**: Specifies the IP address and optional port number where the server will listen. If no port is specified, the default port is used (port 80 for HTTP and port 443 for HTTPS).
            - **Example**: `listen 127.0.0.1:8080;`
            - **IPv6 Example**: `listen [::1]:8080;`

        - **`ssl`**: Enables SSL/TLS for the listening socket. Use this parameter when serving HTTPS traffic.
            - **Example**: `listen 443 ssl;`
            - **Note**: You must also configure SSL certificates using `ssl_certificate` and `ssl_certificate_key`.

        - **`http2`**: Enables HTTP/2 for the specified listening socket. HTTP/2 improves performance with multiplexed streams over a single connection.
            - **Example**: `listen 443 ssl http2;`

        - **`default_server`**: Marks the server as the default for requests that do not match any `server_name`. If there are multiple servers on the same port, the one marked `default_server` will handle unmatched requests.
            - **Example**: `listen 80 default_server;`

    ??? "[`ssl_certificate`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate)"

        Specifies the path to the SSL certificate file used to establish secure HTTPS connections.

        ```{ .lua }
        Syntax:	    ssl_certificate file;
        Default:	—
        Context:	server
        ```

    ??? "[`ssl_certificate_key`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key)"

        Defines the path to the private key file that corresponds to the SSL certificate.

        ```{ .lua }
        Syntax:	    ssl_certificate_key file;
        Default:	—
        Context:	server
        ```

    ??? "[`ssl_protocols`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols)"

        Sets the SSL/TLS protocols that are allowed for secure communication.

        ```{ .lua }
        Syntax:	    ssl_protocols protocol ...;
        Default:	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        Context:	server
        ```


    ??? "[`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream)"

        Is used to define a group of servers (known as an upstream group) for load balancing or failover. You define multiple backend servers, and NGINX distributes requests among them.

        ```{ .lua }
        Syntax:	    upstream name { ... }
        Default:	—
        Context:	http

        upstream backend {
            server backend1.example.com;
            server backend2.example.com;
        }

        server {
            location / {
                proxy_pass http://backend;
            }
        }
        ```


    ??? "[`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)"

        Defines how to handle requests that match specific URI patterns. Can handle different parts of your site (such as static content, dynamic content, API requests, etc.) with separate configurations.

        ```{ .lua}
        Syntax:	    location [ = | ~ | ~* | ^~ ] uri { ... }
                    location @name { ... }
        Default:	—
        Context:	server, location
        ```

        === "`=` (Exact Match)"

            ```{ .lua }
            location = /exact-path {
                <!-- This configuration will only apply to "/exact-path" -->
            }
            ``` 
        === "`~` (Case-Sensitive Regular Expression Match)"

            ```{ .lua }
            location ~ \.php$ {
                <!-- URIs ending in ".php". matches /index.php but not /INDEX.PHP -->
            }
            ``` 
        === "`~*` (Case-Insensitive Regular Expression Match)"

            ```{ .lua }
            location ~* \.(jpg|jpeg|png)$ {
                <!-- URIs ending in (.jpg, .jpeg, .png), case-insensitively -->
            }
            ``` 
        === "`^~` (Prefix Match with Higher Priority)"

            Performs a prefix match, and if this location is matched, NGINX will ignore for other regular expression matches.

            ```{ .lua }
            location ^~ /static/ {
                <!-- This configuration applies to URIs starting with "/static/" -->
            }
            ```
        === "No Modifier (Prefix Match)"

            Checks if the beginning of the URI matches the specified pattern.

            ```{ .lua }
            location /images/ {
                <!-- # This configuration applies to URIs starting with "/images/" -->
            }
            ```

        !!! warning

            When multiple location blocks match a request, NGINX follows this precedence `=`, `^~`, `~ ~*`, No Modifier

    ??? "[`index`](https://nginx.org/en/docs/http/ngx_http_index_module.html#index)"

        ```{ .lua }
        Syntax:	    index file ...;
        Default:	index index.html;
        Context:	http, server, location
        ```

        Defines the default file to serve when a directory is requested.

        Example:
        ```nginx
        index index.html index.php;
        ```

    ??? "[`root`](https://nginx.org/en/docs/http/ngx_http_core_module.html#root)"

        The `root` directive sets the base directory for requests within a specific location.

        ```{ .lua }
        Syntax:	    root path;
        Default:    root html;
        Context:	http, server, location, if in location

        <!-- http://example.com/images/pic.jpg -> /var/www/data/images/pic.jpg -->
        <!-- The path value can contain variables -->
        location /images/ {
            root /var/www/data;
        }
        ```

    ??? "[`alias`](https://nginx.org/en/docs/http/ngx_http_core_module.html#alias)"

        The `alias` directive is used to map a location to a different directory, and it replaces the request URI with the path specified by the `alias`.

        ```{ .lua }
        Syntax:	    alias path;
        Default:	—
        Context:	location

        <!-- http://example.com/images/pic.jpg -> /var/www/data/images/pic.jpg -->
        location /pics/ {
            alias /var/www/data/images;
        }
        ```

        !!! warning

            Choose `alias` when the URI and file system structure differ, and use `root` when they match.

    ??? "[`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)"

        The `proxy_pass` directive sets the backend server to which requests are forwarded.

        ```{ .lua }
        Syntax:	    proxy_pass URL;
        Default:	—
        Context:	http, server, location, if in location

        proxy_pass http://backend;
        proxy_pass http://127.0.0.1:8080;
        proxy_pass http://backend:8080/uri/;
        ```

    ??? "[`proxy_redirect`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_redirect)"

        The `proxy_redirect` directive is used to modify the `Location` and `Refresh` headers in responses from the proxied server, enabling NGINX to rewrite those headers before sending them to the client.

        ```{ .lua }
        Syntax:	    proxy_redirect [default|off] [replacement];
        Default:	—
        Context:	http, server, location

        proxy_redirect default;
        proxy_redirect http://localhost:8000/ http://example.com/;
        proxy_redirect off;
        ```

    ??? "[`proxy_set_header`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)"

        The `proxy_set_header` directive allows you to set or override headers that NGINX sends to the backend server during proxying.

        ```{ .lua }
        Syntax:	    proxy_set_header field value;
        Default:	—
        Context:	http, server, location

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        ```

    ??? "[`proxy_set_body`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_body)"

        The `proxy_set_body` directive defines the content of the body sent to the proxied server. It can be used to modify or remove the request body.
        
        ```{ .lua }
        Syntax:	    proxy_set_body value;
        Default:	proxy_set_body $request_body;
        Context:	http, server, location

        proxy_set_body $request_body;
        proxy_set_body "";
        ```

    ??? "[`return`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return)"

        Immediately sends a specified HTTP response code or redirects the client to a different URL.

        ```{ .lua }
        Syntax:	    return code [text];
                    return code URL;
                    return URL;
        Default:	—
        Context:	server, location, if
        ```

    ??? "[`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#stream)"
    
        ```{ .lua }
        Syntax:     stream { ... }
        Default:    —
        Context:    main

        stream {
            upstream backend {
                server 127.0.0.1:12345;
                server 127.0.0.1:12346;
            }

            server {
                listen 12345;
                proxy_pass backend;
            }
        }

    ??? "[`if`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#if)"

        ```{ .lua }
        Syntax:	    if (condition) { ... }
        Default:	—
        Context:	server, location
        ```

        Evaluates a condition and applies configuration directives based on whether the condition is true.

        - comparison of a variable with a string using the “=” and “!=” operators;
        - regular expression using the “~” (case-sensitive) and “~*” (case-insensitive) operators.
        - checking of a file existence with the “-f” and “!-f” operators;
        - checking of a directory existence with the “-d” and “!-d” operators;
        - checking of a file, directory, or symbolic link existence with the “-e” and “!-e” operators;
        - checking for an executable file with the “-x” and “!-x” operators.

    ??? "[`break`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#break)"

        ```{ .lua }
        Syntax:	    break;
        Default:	—
        Context:	server, location, if
        ```

        Stops the processing of the current `location` block and prevents further rewrite directives from being executed.

        Example:
        ```nginx
        location /images/ {
            if ($request_uri ~* "\.(jpg|png)$") {
                break;
            }
        }
        ```

    ??? "[`rewrite`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite)"

        ```{ .lua }
        Syntax:	    rewrite regex replacement [flag];
        Default:	—
        Context:	server, location, if
        ```

        Changes the request URI based on a pattern and replacement and can redirect or stop the request flow.

        Example:
        ```nginx
        rewrite ^/old-page$ /new-page permanent;
        rewrite ^/product/(.*)$ /product.php?id=$1 last;
        ```

    ??? "[`add_header`](https://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header)"

        Adds custom headers to the HTTP response, which can be used for security, caching, or informational purposes.

        ```{ .lua }
        Syntax:     add_header name value [always];
        Default:    —
        Context:    http, server, location, if in location
        ```



    __Frontend timeouts:__

    ??? "[`client_header_timeout`](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_header_timeout)"

        Defines a timeout for reading client request header. If a client does not transmit the entire header within this time, the request is terminated with the 408

        ```{ .lua }
        Syntax:	    client_header_timeout time;
        Default:	client_header_timeout 60s;
        Context:	http, server
        ```

    ??? "[`client_body_timeout`](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_timeout)"

        Defines a timeout for ___reading client request body___. ==The timeout is set only between two successive read operations, not for the transmission of the whole request body==.

        ```{ .bash }
        Syntax:	    client_body_timeout time;
        Default:	client_body_timeout 60s;
        Context:	http, server, location
        ```

    ??? "[`send_timeout`](https://nginx.org/en/docs/http/ngx_http_core_module.html#send_timeout)"

        Sets a timeout for ___transmitting a response to the client___. ==The timeout is set only between two successive write operations, not for the transmission of the whole response==.

        ```{ .bash }
        Syntax:	    send_timeout time;
        Default:	send_timeout 60s;
        Context:	http, server, location
        ```

    ??? "[`keepalive_timeout`](https://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_timeout)"

        The first parameter sets a timeout during which a keep-alive client connection will stay open on the server side. The zero value disables keep-alive client connections. The optional second parameter sets a value in the “Keep-Alive: timeout=time” response header field. Two parameters may differ.

        ```{ .bash }
        Syntax:	    keepalive_timeout timeout [header_timeout];
        Default:	keepalive_timeout 75s;
        Context:	http, server, location
        ```

    ??? "[`lingering_timeout`](https://nginx.org/en/docs/http/ngx_http_core_module.html#lingering_timeout)"

        This directive specifies the maximum waiting time for more client data to arrive. If data are not received during this time, the connection is closed. Otherwise, the data are read and ignored, and nginx starts waiting for more data again.

        ```{ .bash }
        Syntax:	    lingering_timeout time;
        Default:	lingering_timeout 5s;
        Context:	http, server, location
        ```

    ??? "[`resolver_timeout`](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver_timeout)"

        Sets a timeout for DNS resolution of backend servers.

        ```{ .bash }
        Syntax:	    resolver_timeout time;
        Default:	resolver_timeout 30s;
        Context:	http, server, location
        ```


    __Backend timeouts:__

    ??? "[`proxy_connect_timeout`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_connect_timeout)"

        Defines a timeout for establishing a connection with a proxied server. It should be noted that this timeout cannot usually exceed 75 seconds.

        ```{ .bash }
        Syntax:	    proxy_connect_timeout time;
        Default:	proxy_connect_timeout 60s;
        Context:	http, server, location
        ```

    ??? "[`proxy_send_timeout`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_send_timeout)"

        Sets a timeout for transmitting a request to the proxied server. The timeout is set only between two successive write operations, not for the transmission of the whole request.

        ```{ .bash }
        Syntax:	    proxy_send_timeout time;
        Default:	proxy_send_timeout 60s;
        Context:	http, server, location
        ```

    ??? "[`proxy_read_timeout`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout)"

        Defines a timeout for reading a response from the proxied server. The timeout is set only between two successive read operations, not for the transmission of the whole response.

        ```{ .bash }
        Syntax:	    proxy_read_timeout time;
        Default:	proxy_read_timeout 60s;
        Context:	http, server, location
        ```

    ??? "[`proxy_next_upstream_timeout`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream_timeout)"

        Limits the time during which a request can be passed to the next server. i.e. ==It limits the time spent looping over backend servers, when one timesout== .The 0 value turns off this limitation.

        ```{ .bash }
        Syntax:	    proxy_next_upstream_timeout time;
        Default:	proxy_next_upstream_timeout 0;
        Context:	http, server, location
        ```

    ??? "[`keepalive_timeout`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive_timeout)"

        Sets a timeout during which an idle keepalive connection to an upstream server will stay open.

        ```{ .bash }
        Syntax:	    keepalive_timeout timeout;
        Default:    keepalive_timeout 60s;
        Context:	upstream
        ```

=== "Variables"

    NGINX variables are placeholders that represent values related to the request, response, or the environment. These variables can be used to configure behavior in the NGINX configuration file (nginx.conf), and they change dynamically based on each individual request.

    __Common Variables:__

    - __Request Variables:__

        These variables provide information about the current request being handled.

        - `$host`: The host header from the client’s request or the server name. ex: `example.com`
        - `$uri`: The URI of the request, without any query string. ex: `/about?user=123`
        - `$request_uri`: The full original request URI, including query string. ex: `/about?user=123`
        - `$args`: The query string (i.e., everything after the ? in the URI). ex: `user=123&status=active`
        - `$request_method`: The HTTP method used in the request. ex: `GET, POST, PUT`
        - `$request_body`: The request body
        - `$remote_addr`: The IP address of the client making the request. ex: `192.168.1.10`
        - `$remote_port`: The port number of the client. ex: `50234`
        - `$server_addr`: The IP address of the server handling the request. ex: `192.168.1.100`
        - `$server_port`: The port number on which the request was received. ex: `80, 443`

    - __Response Variables:__

        These variables relate to the response the server is sending back.

        - `$status`: The HTTP status code of the response. ex: `200, 404`

    - __HTTP Header Variables:__

        ==You can access HTTP headers by prepending http_ to the header name (with hyphens replaced by underscores)==.

        - `$http_user_agent`: The client’s User-Agent header. ex: `Mozilla/5.0 (Windows NT 10.0; Win64; x64)`
        - `$http_referer`: The referring URL (the page the request came from). ex: `https://example.com/previous-page`
        - `$http_token`: 

    - __Cookie Variables:__

        ==You can access cookies by prepending cookie_ to the cookie name==.

        - `$cookie_session_id`: Accessing request cookie with key `session_id`.

    - __Custom Variables:__

        You can define your own variables in the NGINX configuration using the set directive.

        ```{ .lua }
        set $my_variable "custom_value";

        server {
            location / {
                return 200 "$my_variable";
            }
        }
        ```