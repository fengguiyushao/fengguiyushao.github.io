---
layout: default_post
title:  windows自动切换代理
---

### windows平台代理设置

在google上查资料需要设置代理，但通过代理访问国内网站时速度较慢，所以需要能根据访问的网站自动切换代理。

### PAC规则

代理自动配置（英语：Proxy auto-config，简称PAC）是一种网页浏览器技术，用于定义浏览器该如何自动选择适当的代理服务器来访问一个网址。

一个PAC文件包含一个JavaScript形式的函数`FindProxyForURL(url, host)`。这个函数返回一个包含一个或多个访问规则的字符串。用户代理根据这些规则适用一个特定的代理其或者直接访问。 当一个代理服务器无法响应的时候，多个访问规则提供了其他的后备访问方法。

浏览器在访问其他页面以前，首先访问这个PAC文件。PAC文件中的URL可能是手工配置的，也可能是是通过网页的网络代理自发现协议（Web Proxy Autodiscovery Protocol）自动配置的。

 1. /twitter.com/i 以正则匹配访问的 URL
 2. 192.168.*.* 匹配目标 IP 为 192.168 这个网段
 3. 192.168/16 匹配目标 IP 为 192.168 这个网段
 4. *google.com 通配符匹配的域名
 5. .google.com 匹配所有以 .google.com 结尾的域名
 6. www.google.com 匹配域名为 www.google.com
 7. useragent:Twitter 请求 PAC 时 UserAgent 包含 Twitter，返回的 PAC 将会只含指定代理
 8. 其他包含通配符 * 的字符串 使用通配符匹配访问的 URL
 9. 不包含 * 的字符串 直接匹配访问的 URL
 
### PAC语法
FindProxyForURL函数有三种可能的返回值，一是`DIRECT`,就是直接连接，不通过代理；二是`PROXY proxyaddr:port`，其中proxtaddr和port分别是代理的地址和代理的端口；三是`SOCKS socksaddr:port`，其中socksaddr和port分别是socks代理的地址和端口，一个自动代理文件可以是多个选择的组合，其中用分号(;)隔开，如

```
function FindProxyForURL(url,host){
    if(host=="www.robinzhou.com")
        return "DIRECT";
    return "PROXY myproxy:80;
    PROXY myotherproxy:8080;
    DIRECT";
}
```

现在我们介绍下这个proxy.pac脚本文件,脚本的语法是js语法,js的内置函数可以使用,要实现自动配置代理,需要实现`FindProxyForURL`这个函数,其参数url代表要访问的连接,host代表要访问连接的主机名,该函数有三个返回参数( direct:直接连接,proxy IP:PORT,socket IP:PORT), 返回结果大小写不敏感．

下面我们介绍几个常用的PAC函数，并举便说明：
#### 1 `isPlainHostName(host)`，判断是否为本地主机，例如以http://myservername/的方式访问，则是直接连接，否则使用代理

```
        function FindProxyForURL(url, host)
        {
          if (isPlainHostName(host))
            return "DIRECT";
          else
            return "PROXY proxy:80";
        }
```

#### 2 `dnsDomainIs(host, "")`、`localHostOrDomainIs(host, "")`，判断所访问主机是否属于某个域和某个域名,例如属于.company.com域的主机名，www.company.com和home.company.com的直接连接，否则使用代理访问。

```
         function FindProxyForURL(url, host)
        {
          if ((isPlainHostName(host) ||
             dnsDomainIs(host, ".company.com")) &&
            !localHostOrDomainIs(host, "www.company.com") &&
            !localHostOrDomainIs(host, "home.company.com"))
            return "DIRECT";
          else
            return "PROXY proxy:80";
        }
```

#### 3 `isResolvable(host)`，判断被访问主机名能否被解析．例子演示主机名能否被dns服务器解析，如果能直接访问，否则就通过代理访问。
```
        function FindProxyForURL(url, host)
        {
          if (isResolvable(host))
            return "DIRECT";
          else
            return "PROXY proxy:80";
        }
```        
#### 4 `isInNet(host, "", "")`，判断IP是否在某个子网内．例子演示访问IP段的主页不使用代理。
```
      function FindProxyForURL(url, host)
      {
          if (isInNet(host, "10.0.0.0", "255.255.0.0"))
            return "DIRECT";
          else
            return "PROXY proxy:80";
      }
```

#### 5 `shExpMatch(host, "")`，判断被访问主机名是否符合某一正则表达式．本例演示根据主机域名来改变连接类型，本地主机、*.edu、*.com分别用不同的连接方式。

```
        function FindProxyForURL(url, host)
        {
          if (isPlainHostName(host))
            return "DIRECT";
          else if (shExpMatch(host, "*.com"))
            return "PROXY comproxy:80";
          else if (shExpMatch(host, "*.edu"))
            return "PROXY eduproxy:80";
          else
            return "PROXY proxy:80";
        }
```

#### 6 `rl.substring()`，取URL字符串的子串．本例演示根据不同的协议来选择不同的代理，http、https、ftp、gopher分别使用不同的代理。
```
        function FindProxyForURL(url, host)
        {
            if (url.substring(0, 5) == "http:") {
              return "PROXY proxy:80";
            }
            else if (url.substring(0, 4) == "ftp:") {
              return "PROXY fproxy:80";
            }
            else if (url.substring(0, 7) == "gopher:") {
              return "PROXY gproxy";
            }
            else if (url.substring(0, 6) == "https:") {
              return "PROXY secproxy:8080";
            }
            else {
              return "DIRECT";
            }
        }
```
#### 7 `dnsResolve(host)`，解析地址．本例演示判断访问主机是否某个IP，如果是就使用代理，否则直接连接。
```
      function FindProxyForURL(url, host)
      {
         if (dnsResolve(host) == "166.111.8.237") {
              return "PROXY secproxy:8080";
            }
            else {
              return "PROXY proxy:80";
            }
      }
```
#### 8 `myIpAddress()`，返回自己的IP地址．本例演示判断本地IP是否某个IP，如果是就使用代理，否则直接使用连接。

```
        function FindProxyForURL(url, host)
        {
            if (myIpAddress() == "10.1.1.1") {
              return "PROXY proxy:80";
            }
            else {
              return "DIRECT";
            }
        }
```

#### 9 `dnsDomainLevels(host)`，返回被访问主机域名级数．本例演示访问主机的域名级数是几级，就是域名有几个点，如果域名中有点，就通过代理访问，否则直接连接。

```
        function FindProxyForURL(url, host)
        {
            if (dnsDomainLevels(host) > 0) { // if number of dots in host > 0
              return "PROXY proxy:80";
            }
             return "DIRECT";
        }
```

#### 10 `weekdayRange()`，判断当前日期日否在某一星期段．本例演示当前日期的范围来改变使用代理，如果是GMT时间周三到周六，使用代理连接，否则直接连接。

	function FindProxyForURL(url, host)
	{
		if(weekdayRange("WED", "SAT", "GMT"))
			return "PROXY proxy:80";
		else
			return "DIRECT";
	}


#### 11 最后一个例子是演示随机使用代理，这样可以好好利用代理服务器。

<!--?prettify autoload=true?-->
```
      function FindProxyForURL(url,host)
      {
            return randomProxy();
      }
      function randomProxy()
      {
           switch( Math.floor( Math.random() * 5 ) )
           {
               case 0:
                   return "PROXY proxy1:80";
                   break;
               case 1:
                   return "PROXY proxy2:80";
                   break;
               case 2:
                   return "PROXY proxy3:80";
                   break;
               case 3:
                   return "PROXY proxy4:80";
                   break;
               case 4:
                   return "PROXY proxy5:80";
                   break;
           }   
      }
```

### gfwlist转换成PAC文件

提供3种方法直接生成PAC文件

 1. SwitchSharp里导出
 2. 从站点[AutoProxy2PAC](https://autoproxy2pac.appspot.com/)定制下载或者直接引用
 3. 使用工具将gfwlist转换成PAC，如[JinnLynn/GenPAC](https://github.com/JinnLynn/GenPAC)