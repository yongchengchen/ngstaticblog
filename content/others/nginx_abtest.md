<!--
Categories = ["Development", "Others"]
Description = ""
Tags = ["Development", "Nginx"]
date = "2016-11-16T10:47:31-08:00"
menu = "main"
title = "Nginx setting for A/B Testing container"
-->

## Nginx setting for A/B Testing container

The solution is based on browser cookie "ab_test".

If user first time to access the website, we will randomly dispatch to the backend server, then we set cookie value with the server type(main/test), then for the next request, we will check the cookie value, and dispath it to the previous server.


### 1. define constants

```shell
map $host $MAIN_SERVER { default 127.0.0.2; }
map $host $TEST_SERVER { default 127.0.0.3; }
```

### 2. retrive some variables such as client cookie

```shell
map $http_user_agent $mobile_agent{
    default 0;
    ~*(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge\s|maemo|midp|mmp|netfront|opera\sm(ob|in)i|palm(\sos)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows\s(ce|phone)|xda|xiino 1;
}

split_clients "app${remote_addr}${http_user_agent}${date_gmt}" $ab_test_token {
        100% "main";
        * "test";
}

map $cookie_ABTest $ABTEST_COOKIE_VALUE {
        default    $ab_test_token;
        "test"     "test";
        "main" "main";
}

map "${ABTEST_COOKIE_VALUE}" $server_host {
        default $MAIN_SERVER;
        "main"  $MAIN_SERVER;
        "test"  $TEST_SERVER;
}

map $https $req_port {
        default 80;
        "on"  443;
}

map $ABTEST_COOKIE_VALUE $cookie_expires {
        default 1;
}

```

### 3. define proxy gateway

```shell
server {
        server_name youservername.here;

        listen 80;
        listen 443 ssl;

        #ssl setting begin
        ssl_certificate     /path/to/cert/file.crt;
        ssl_certificate_key /path/to/cert/file.key;
        ssl_protocols        TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        #ssl setting end;

        #get real ip begin
        set_real_ip_from  172.31.0.0/16;
        set_real_ip_from  127.0.0.1/32;
        real_ip_header    X-Forwarded-For;
        real_ip_recursive on;

        set $real_x_forward_for $HTTP_X_FORWARDED_FOR;
        if ($real_x_forward_for = "" ) {
                set $real_x_forward_for $proxy_add_x_forwarded_for;
        }
        #get real ip end

        # get domain name without "www" begin
        set $domain $host;
        if ($domain ~ "www\.(.*)" ) {
                set $domain $1;
        }
        # get domain name end

        location / {
                proxy_redirect off;
                proxy_cache    off;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $real_x_forward_for;
                proxy_set_header X-Forwarded-Ssl $https;
                add_header       Front-End-Https $https;
                add_header       Set-Cookie "ABTest=${ABTEST_COOKIE_VALUE};Path=/;Max-Age=${cookie_expires};Domain=.${domain};";
                proxy_pass $scheme://$server_host:$server_port;
        }
}
```

### 4. set main server 

```shell
server {
        listen 127.0.0.2:80;
        listen 127.0.0.2:443 ssl;

        #ssl setting begin
        ssl_certificate     /path/to/cert/file.crt;
        ssl_certificate_key /path/to/cert/file.key;
        ssl_protocols        TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        #ssl setting end;

        set_real_ip_from  127.0.0.1/32;
        set_real_ip_from  172.31.0.0/16;
        real_ip_header    X-Forwarded-For;
        real_ip_recursive on;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
            #limit_req   zone=one  burst=1 nodelay;
        }
}
```

### 5. set test server

```php
server {
        listen 127.0.0.3:80;
        listen 127.0.0.3:443 ssl;

        #ssl setting begin
        ssl_certificate     /path/to/cert/file.crt;
        ssl_certificate_key /path/to/cert/file.key;
        ssl_protocols        TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        #ssl setting end;

        set_real_ip_from  127.0.0.1/32;
        set_real_ip_from  172.31.0.0/16;
        real_ip_header    X-Forwarded-For;
        real_ip_recursive on;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
        limit_req   zone=one  burst=1 nodelay;
        }
}
```
