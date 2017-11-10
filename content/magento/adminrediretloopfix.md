<!--
Categories = ["Magento"]
Description = "Fixing Magneto2 admin url redirect loop"
Tags = ["Development", "Magento 2"]
date = "2016-10-23T21:47:31-08:00"
title = "Fixing Magneto2 admin url redirect loop"
-->

### Fixing Magneto2 admin url redirect loop

#### I'm using docker group as magento2 development enviroment. 

And when I tried to login admin pannel, I got a redirect loop.

And I dig into the source code, I found the problem is located in the class 

```php
Magento\Backend\Controller\Adminhtml\Auth\Login

public function execute() { 
    if ($this->_auth->isLoggedIn()) { 
        if ($this->_auth->getAuthStorage()->isFirstPageAfterLogin()) { 
            $this->_auth->getAuthStorage()->setIsFirstPageAfterLogin(true); 
        } 
        return $this->getRedirect($this->_backendUrl->getStartupPageUrl()); 
    } 

    $requestUrl = $this->getRequest()->getUri(); 
    $backendUrl = $this->getUrl('*'); 
    // redirect according to rewrite rule 
    if ($requestUrl != $backendUrl) { 
        return $this->getRedirect($backendUrl); 
    } else { 
        return $this->resultPageFactory->create(); 
    } 

```

I'm running two docker container, one is Nginx proxy server, the other one is magento nginx+php-fpm server.
All requests go to Nginx proxy server and then proxy pass to real magento server.
Then I checked my nginx proxy server configuration, I found the problem!

```php
...
resolver 127.0.0.1; 
proxy_set_header Host $host:$port; 
...
```

After changing as code below, the problem is fixed:

```php

...
resolver 127.0.0.1; 
proxy_set_header Host $host; 
...
```