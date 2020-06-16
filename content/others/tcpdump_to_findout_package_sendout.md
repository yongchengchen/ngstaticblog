<!--
Categories = ["Development", "Others"]
Description = ""
Tags = ["Development", "redis", "redis-cluster", "redis-shake", "redis-proxy", "Others"]
date = "2020-06-16T21:57:31-08:00"
title = "Development Tips"
-->

## Redis cluster and migrate data from standalone redis

Just got an email from AWS about an ec2 instance has been implicated in activity which resembles attempts to access remote hosts on the internet without authorization.

First I checked the instance security group setting, firewall settings, they all looks good. So what's going on?


1. confirm the problem exists
run command to check the packages been sent out.

```
sar -n DEV 2 10 
```

and yes, there are a lot of net packages send out.

2. use tcpdump -nn

```
sudo tcpdump -nn
```

Since the server there are a lot of docker instances runing. so not sure if there any docker instance has been compromised.

Luckly all these docker instance are testing env, so just stop the docker server.

Then I found there are still a lot of package been sent out.


3. use nethogs to identify which process send out package.
Install nethogs and start it.

``` 
sudo nethogs <ethinterface>
```

**If docker service is not stoped, nethogs will have error like 'could not open proc ....'**

Finally I found it's because of a nginx server upstream to test server since I put a wildcard upstream. which is a vulnerability.


Be careful setting Nginx upstream to a default fallback proxy to rules.

