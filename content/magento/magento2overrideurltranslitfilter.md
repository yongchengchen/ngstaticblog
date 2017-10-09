<!--
Categories = ["Development", "Magento"]
Description = ""
Tags = ["Development", "Magento"]
date = "2015-12-12T21:47:31-08:00"
title = "Magento2 Let Category Name support last character +"
-->

## Magento2 Let Category Name support last character +

The website I just developed which has some categories name the last character is "+", and when we import product from CSV, it will automatically removed by Magento. 

Normally it's fine until we found issue below:

We have two categories:
```shell
1. CategoryName
2. CategoryName+
```
So we have to fix it.

Just simply using plugin to correct it and convert "+" as "-plus".

## Define the plugin class

```php
<?php

namespace Your\ModuleName\Plugin\Framework\Filter;

class TranslitUrl
{
    public function aroundFilter($origin, $process, $string)
    {
        $p1 = substr($string, 0, -1);
        $p2 = substr($string, -1);

        if ($p2 === '+') {
            $p2 = '-plus';
        }

        if (strpos($string, 'AJ1800') > 0) {
             echo "$string => $p1 . $p2";
        }

        $string = $p1 . $p2;

        return $process($string);
    }
}
```

## Add it to plugin settings "etc/di.xml"
```xml
    <type name="Magento\Framework\Filter\TranslitUrl">
        <plugin name="urltranslit_fix"
                type="Your\ModuleName\Plugin\Framework\Filter\TranslitUrl"
                sortOrder="1" />
    </type>
```

