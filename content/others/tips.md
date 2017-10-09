<!--
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2016-10-23T21:47:31-08:00"
title = "Development Tips"
-->

## useful tips
* grep get ip addr:
```shell
grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}"
```
* unique grep result:
```shell
grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" | cut -d ' ' -f 4 | sort -u
```
* inotify:

  install 
```shell 
 yum --enablerepo=epel -y install inotify-tools 
```
 monitor.sh 
```shell
#!/bin/bash
TARGET=$1
ACTION=$2
while inotifywait -e modify ${TARGET}; do
        /bin/bash ${ACTION} ${TARGET}
done
```

action.sh
```shell
#!/bin/bash
#/var/log/nginx/error.log

TARGET=$1
cat ${TARGET} | grep 'limiting requests' | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" | cut -d ' ' -f 4 | sort -u
```

use it:
```shell
./monitor.sh /var/log/nginx/error.log ./action.sh
```

