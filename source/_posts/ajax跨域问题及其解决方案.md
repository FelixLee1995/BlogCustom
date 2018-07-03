---
title: ajax跨域问题及其解决方案
date: 2018-05-03 12:40:02
tags: ajax
categories: 技术
---

ajax跨域问题是经常遇到的了，这次总结一下问题出现的原因和解决方案。

什么情况下会出现跨域呢？

 1. 浏览器处于安全因素，对请求进行了限制（极有可能请求是200的）
 2. 发出的请求是XHR请求，即XMLHttpRequest，当我们请求的是script类型或者是其他静态资源时，不一定会产生跨域问题。
 3. 请求的资源是跨域的，即协议，域名，端口，三者任一不一致时，才满足跨域的基本条件。

针对浏览器的限制，我们可以使用命令行启动来解决，chrome可以直接 
> --disable-web-security

第二种解决方案是，使用jsonp来请求，这时候，前台ajax里规定好对应的回调函数，即可对应解析出后台返回的包装好的js代码。

```javascript
$.ajax({
		type: "get",
		url: base+"/getJsonp",
		dataType: "jsonp",
		jsonp: "callback", //默认是callback，和后台对应
		success: function(data) {
			console.log(data);
		}
	});
```

但是jsonp有很大的弊端。首先是服务器需要改动，若是spring框架，需要增加AbstractJsonpResponseBodyAdvice的切片，规定回调函数的名称，若是第三方的api，GG。其次是只支持GET的请求和jsonp的数据类型。可以说，jsonp的方式，适用于后台可控的情况。而且代码需要配合改动。

那么在被调用方，还有什么方式可以快捷地支持跨域呢？

 1. Filter. 应用服务器通过增加filter来在 response header里加入跨域的支持。

```java
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HttpServletResponse res = (HttpServletResponse) response;
		//让所有origin域名都支持跨域
		HttpServletRequest req = (HttpServletRequest) request;
		
		String origin = req.getHeader("origin");
		if (!org.springframework.util.StringUtils.isEmpty(origin)) {
			res.addHeader("Access-Control-Allow-Origin", origin);
		}
		
		//让所有header都支持跨域
		String header = req.getHeader("Access-Control-Request-Headers");
		if (!org.springframework.util.StringUtils.isEmpty(header)) {
			res.addHeader("Access-Control-Allow-Headers", header);
		}
		
		res.addHeader("Access-Control-Allow-Methods", "GET");
		res.addHeader("Access-Control-Allow-Credentials", "true");
		res.addHeader("Access-Control-Max-Age", "3600");
		
		chain.doFilter(request, response);
		
	}
```

这样的话浏览器在接受到response之后，在header里面读到允许跨域的范围信息，即可支持跨域。（若是spring框架，更快捷的方式是在controller的 类或者方法上加上 ***@CrossOrigin*** 注解）

如果不想修改应用服务器呢？还可以在静态服务器上设置好参数，是一样的作用。以Nginx为例：

```java
location ~ /{
    add_header Access-Control-Allow-Origin $http_origin;
    add_header Access-Control-Allow-Headers $http_access_control_request_headers;
    add_header Access-Control-Allow-Methods *;
    add_header Access-Control-Max-Age 3600;
    add_header Access-Control-Allow-Credentials true;
  }
```
在对应vhost的配置文件里加入跨域支持，reload，即可支持跨域。

终极的解决方案，是我们可以直接配置nginx反向代理，把所有的请求路径代理到nginx服务器上。

**关于Nginx的学习，待续！**


