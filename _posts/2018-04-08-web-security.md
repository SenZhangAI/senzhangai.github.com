---
layout: post
title: "网络安全"
description: ""
keywords: web security
category: web
tags: [web]
---

## 前言
总结 web 安全相关知识点,

参考资料：

* <http://guides.rubyonrails.org/security.html>
* <http://www.owasp.org.cn>
* <https://www.aliyun.com/product/waf>



## Sessions
http是无状态的，而用户在进行一系列web服务操作时往往需要保存之前的状态，cookies为服务器保存在客户端的状态，
而sessions则是用户登录后保存在服务器端的状态。
通常cookie中也保存一个例如取名`sessions_id`的数字字符串用来标识服务器端记录的用户状态。

### Sessions-ID

首先确保Sessions-ID 不能轻易地被暴力破解

### Sessions劫持

显然如果用户Sessions_id被截获了，就可以欺骗服务器已该用户身份登录，
所以要防止被嗅探，例如未加密的WiFi很容易被嗅探，解决方法:

1. 加密网络，例如https,
2. 用户界面设计上应有log-out功能清除cookie
3. 许多跨域脚本(XSS, cross-site scripting)目标是获取用户cookie，注意防范
4. 防止固定Session ID攻击

### Sessions Guidelines

1. 重要的数据不应保存在session中以免丢失，因为session只是临时的会话，用户logout时（而不是关闭浏览器时）清除session，同时session也有失效时间，以免越积越多。
清除session不同的框架应该有不同的处理策略，有直接从内存中清除的，有记录在硬盘上再取的等等。
2. 大的Objects不应保存在session中，而应保存在数据库里，Session中保存这些Objects的Id，一方面如果修改了Objects的结构，数据在硬盘里没有丢失，容易兼容原有的session，另一方面也避免暂用过多内存。

### Session Storage
Cookies限制4kB，不应存大量数据
cookies对用户而言可见，所以重要数据应加密，秘钥应妥善保管

### 重放攻击(Replay Attacks)

原理是劫持信息者虽然不知道加密的内容，但可以通过重复发送，
例如，假设一台银联刷卡机，用户请求一次，但重放多次
(更常见的例子应该是流量攻击或者多次post注入同样的数据到数据库等)。

通过使用timestamp和nonce(number once, 只使用一次的随机数)来做的重放机制。

timestamp用来判断同一个请求上一次的请求时间，不仅仅可用于重放攻击，还可防止网络延迟下多次请求。即防止短时间内重复请求。

当然，还的判断过一段较长时间是否为重复请求，可以生成一个一次性的随机数标记(nonce)，附加到请求中，如果服务器接受请求后发现已存在该随机数，极有可能是重复请求。

### Session Fixation(固定Session攻击)

如果用户在Server上的Session Id是固定的，有可能被利用，

攻击者用自己的账号登录，获取自己的session id，例如:`_session_id=abc`,并不断刷新防止session id失效

例如跨域脚本等手段当用户的session id设置为攻击者自己的

```javascript
<script>document.cookie="_session_id=abc";</script>
```

这样用户和攻击者将共用同一个session，这样充钱之类的就很危险

解决方法是每次登录后设置新的Session id并确保**原session id失效**,

另一种方法是在session中保存用户的某种信息用于辨识该用户（例如IP，或者agent name），但agent name 容易伪装，如果用IP地址辨识，因为用户也有可能用代理或者ISP代理导致问题，所以不是一个好办法

### Session失效

注意例如Fixation Session攻击中攻击者可能通过不断刷新延后Session的失效时间，
所以失效时间不能仅根据更新时间，还应设置created_at，当超过例如2天后也失效

```ruby
delete_all "updated_at < '#{1.hour.ago.to_s(:db)}' OR
            created_at < '#{2.days.ago.to_s(:db)}'"
```

## Cross-Site Request Forgery (CSRF)跨域请求伪造

原理：

假设用户登录A网站`www.A.com`，而A网站中设计了一个请求：

```html
<img src="http://www.B.com/project/1/destroy">
```

而恰巧用户时网站B的管理员，拥有其管理员相关cookies，
而浏览器在发送给域`B.com`请求时会自动将相关的cookies附上，
因此就拥有了操作B网站的权限。

CSRF是非常重要的安全问题。

对策：

首先正确地使用GET和POST(还有其他的请求类型，但有些浏览器不支持)

```
If your web application is RESTful, you might be used to additional HTTP verbs, such as PATCH, PUT or DELETE. Most of today's web browsers, however, do not support them - only GET and POST. Rails uses a hidden _method field to handle this barrier.
```
