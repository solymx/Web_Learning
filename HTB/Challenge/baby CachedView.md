## 考點

- DNS Rebinding attack
- 繞過 ip 位址限制


## 作法

題目敘述是說做了一個幫你 cache 網站的服務，然後有對 ip 做限制

抓網站程式碼下來

看 `requirements.txt`，所以是 flask
```
Flask
selenium
```

看 routes.py ，擷取部分
從下面知道 flag 路徑
```
@web.route('/flag')
@is_from_localhost
def flag():
    return send_file('flag.png')
```

在 routes.py 可以知道有用到 util.py ，所以看一下  util.py
這邊擷取片段
```
def cache_web(url):
    scheme = urlparse(url).scheme
    domain = urlparse(url).hostname

    if not domain or not scheme:
        return flash(f'Malformed url {url}', 'danger')
        
    if scheme not in ['http', 'https']:
        return flash('Invalid scheme', 'danger')

    def ip2long(ip_addr):
        return struct.unpack('!L', socket.inet_aton(ip_addr))[0]
    
    def is_inner_ipaddress(ip):
        ip = ip2long(ip)
        return ip2long('127.0.0.0') >> 24 == ip >> 24 or \
                ip2long('10.0.0.0') >> 24 == ip >> 24 or \
                ip2long('172.16.0.0') >> 20 == ip >> 20 or \
                ip2long('192.168.0.0') >> 16 == ip >> 16 or \
                ip2long('0.0.0.0') >> 24 == ip >> 24
    
    if is_inner_ipaddress(socket.gethostbyname(domain)):
        return flash('IP not allowed', 'danger')
    
    return serve_screenshot_from(url, domain)

def is_from_localhost(func):
    @functools.wraps(func)
    def check_ip(*args, **kwargs):
        if request.remote_addr != '127.0.0.1' or request.referrer:
            return abort(403)
        return func(*args, **kwargs)
    return check_ip
```

上面可以看到有限制 ip 位址


如果在輸入框輸入：`http://167.172.52.221:31069/flag`
會因為 `check_ip` 的檢查被禁止

如果在輸入框輸入：`http://127.0.0.1:31069/flag`
會因為 `cache_web` 的檢查被禁止


check_ip 這個繞不過去的話，那要符合 cache_web 的檢查就是給一個正常的網站

然後那個網站會跳轉到 `127.0.0.1/flag`

所以建立一個 index.html ，寫入底下
```
<meta http-equiv="refresh" content="0; URL=http://127.0.0.1/flag" />
```
(參考：https://skelter.hashnode.dev/htb-baby-cachedview-writeup-alternative)

我把這個網站直接架設在 github pages
架設參考：https://medium.com/%E9%80%B2%E6%93%8A%E7%9A%84-git-git-git/%E5%BE%9E%E9%9B%B6%E9%96%8B%E5%A7%8B-%E7%94%A8github-pages-%E4%B8%8A%E5%82%B3%E9%9D%9C%E6%85%8B%E7%B6%B2%E7%AB%99-fa2ae83e6276

之後讓這題目去瀏覽自己的 github pages 即可


## 參考

- [[HTB]一道涉及DNS Rebinding attack的题(未做出来)](https://1dayluo.github.io/post/htbyi-dao-she-ji-dns-rebinding-attack-de-ti-wei-zuo-chu-lai/)
- [[HTB] Baby CachedView writeup (Alternative)](https://skelter.hashnode.dev/htb-baby-cachedview-writeup-alternative)