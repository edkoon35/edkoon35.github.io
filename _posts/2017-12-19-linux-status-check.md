---
title: "Nginx L7 healthCheck 확인 스크립트"
date: 2017-08-04 11:13:09
tags:
- linux
- script
- healthcheck
---
# Desc
Nginx L7 healthCheck 확인 스크립트

```
host=$(hostname)

alias l7chk="tail -f /PATH/NGINX.log | awk '{
                if (\$6 ~ \"200\" && \$8 ~ \"l7.html\")
                    print \"[$host] : Open Code \" \$6
                else if (\$6 ~ \"404\" && \$8 ~ \"l7.html\")
                    print \"[$host] : Block Code \" \$6
            }'"
```
