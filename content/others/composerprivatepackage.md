<!--
Categories = ["Development", "Others"]
Description = "Some time your will not want your company's code goes to public project, but you want composer to manage packages.
Here's an example for how to let composer use private packages, and put these packages to customize folder."
Tags = ["Development", "Composer"]
date = "2016-11-25T21:47:31-08:00"
menu = "main"
title = "Composer use private git packages"
-->

## Composer use private git packages

* Why doing this?

Some time your will not want your company's code goes to public project, but you want composer to manage packages.

Here's an example for how to let composer use private packages, and put these packages to customize folder.

* 1. Create mycomposer.php to do these automaticly

Here we use laravel as an example.

```php
<?php

$now = strtotime('now');
$tmp_work_folder = sprintf('/tmp/mycomposer_%s', $now);
$current_folder = __DIR__;
$composer_json = $current_folder . '/composer.json';
$mycomposer_json = $current_folder . '/mycomposer.json';

if(!file_exists($composer_json)) {
    rename($current_folder, $tmp_work_folder);
    exec('composer create-project --prefer-dist laravel/laravel ' . $current_folder);
    exec(sprintf('cp -a %s/* %s', $tmp_work_folder, $current_folder));
}

if (file_exists($mycomposer_json)) {
    $composer_configs = json_decode(file_get_contents($composer_json), true);
    $mycomposer_configs = json_decode(file_get_contents($mycomposer_json), true);

    $requires = [];
    foreach( $mycomposer_configs['repositories'] as $package_config) {
        $requires[$package_config['package']['name']] = '*';
    }
    $mycomposer_configs['require'] = $requires;

    $new_composer_configs = array_replace_recursive($composer_configs, $mycomposer_configs);
    $new_composer_configs['repositories'] = $mycomposer_configs['repositories'];
    file_put_contents($composer_json, json_encode($new_composer_configs, JSON_PRETTY_PRINT));

    chdir($current_folder);
    exec("cd $current_folder && composer update");
}
```

* 2. Create a JSON file name it as mycomposer.json

```php
{
        "repositories": [
            {
                "type": "package",
                "package": {
                    "name": "../your/path/Module1",
                    "version":"0.0.0",
                    "source": {
                        "url": "https://url_to_your_git_repostory1.git",
                        "type": "git",
                        "reference": "master"
                    }
                }
            },
            {
                "type": "package",
                "package": {
                    "name": "../your/path/Module2",
                    "version":"0.0.0",
                    "source": {
                        "url": "https://url_to_your_git_repostory2.git",
                        "type": "git",
                        "reference": "master"
                    }
                }
            }
        ],

        "autoload": {
             "psr-4": {
                    "YourNameSpace\\": "your/path"
                }
            }
        },

        "require": {
        },

        "require-dev": {
        }
}
```
* 3. put mycomposer.php and mycomposer.json to your new project path and run the command:

```shell
php mycomposer.php
```

it's done!
