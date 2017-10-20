<!--
Categories = ["Development", "Others"]
Description = ""
Tags = ["Development", "shell"]
date = "2016-11-16T10:47:31-08:00"
menu = "main"
title = "A wrapper for cron job log"
-->

## A wrapper for cron job log

When we setup a cron job, we redirect log to a /tmp/path file. But sometimes the /tmp/path will be delete by system. 

So I wrote a very simple script to make sure the directory will be created, and the file will be sync to s3 automaticlly.
 
### 1. script

```shell
#!/bin/bash
CMD=$1
IFS='>' read -ra PARTS <<< "$CMD"    #Convert string to array

#Print all names from array
LOGPART=''
for i in "${PARTS[@]}"; do
    LOGPART=$i
done

if [ $LOGPART != '/dev/null' ]; then
        if [ ! -f $LOGPART ]; then
                mkdir -p $LOGPART
                rm -rf $LOGPART
        fi
fi
eval $CMD

LOGPART=`echo "${LOGPART}" | sed -e 's/^[ \t]*//'`
S3Backup="aws s3 cp ${LOGPART} s3://urlpath/log${LOGPART} --profile logiam > /dev/null"
eval $S3Backup
```

### 2. how to use

```shell
* * * * * cronrunner.sh "php yourfile.php params >> /tmp/log/log1/logs.txt"
```
