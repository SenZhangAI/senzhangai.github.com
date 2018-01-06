---
layout: post
title: "REST API返回错误码的方案"
description: "REST API error code"
keywords: rest, restful, api, error code
category: Programming
tags: [rest, api, design]
---

## 2018.1.6 更新
本文对于RESTful理解还不透彻，后来才了解到http的错误码是应用层，作为RESTful错误码映射挺好，
至于可能与中间代理的错误码混淆的问题，这个可以在返回中再加自定义的特殊的错误码解决，撞错误码的概率相对较低。
RESTful 的error code规则就是大家都遵循的一种规范，方便大家理解api，实践中应该也很好用，我的设计也是个差不多的方案，但没必要再设计个规则。

## 争论
关于RESTful API 错误码到底如何返回，存在一些争论，
有的认为直接用http的错误码，
有的认为http与rest分别属于传输层与应用层，需区分开来，只要传输成功统一返回200，
如果REST API出了问题，再返回例如`err_code`
这样便于判断到底是传输层出了问题，还是应用层出了问题。
各有各的道理，我整理一些有用的信息。

## 支持http错误码与REST api错误码分离的观点

参见： <https://stackoverflow.com/a/46379701>

该回答虽然赞同数比较少，但答案较新，我觉得有一定的道理，
其观点重点在于传输层与应用层错误混合在一起对于错误判断来说容易引起混淆。

摘录如下：

Ugh... (309, 400, 403, 409, 415, 422)... a lot of answers trying to guess, argue and standardize what is the best return code for a successful http request but a failed rest call.

It is **wrong** to mix HTTP protocol codes and REST results.

However, I saw many implementations mixing them, and many developers may not agree with me.

HTTP return codes are related to the `HTTP Request` itself. A REST call is done using a Hypertext Transfer Protocol request and it works at a lower level than invoked REST method itself. REST is a concept/approach, and its output is a business/logical result, while HTTP result code is a transport one.

For example, returning "404 Not found" when you call /users/ is confuse, because it may mean:

* URI is wrong (HTTP)
* No users are found (REST)

"403 Forbidden/Access Denied" may mean:

* Special permission needed. Browsers can handle it by asking the user/password. (HTTP)
* Wrong access permissions configured on the server. (HTTP)
* You need to be authenticated (REST)

And the list may continue with '500 Server error" (an Apache/Nginx HTTP thrown error or a business constraint error in REST) or other HTTP errors etc...

From the code, it's hard to understand what was the failure reason, a HTTP (transport) failure or a REST (logical) failure.

If the HTTP request physically was performed successfully it should always return 200 code, regardless is the record(s) found or not. Because URI resource is found and was handled by the http server. Yes, it may return an empty set. Is it possible to receive an empty web-page with 200 as http result, right?

Instead of this you may return 200 HTTP code with some options:

* "error" object in JSON result if something goes wrong
* Empty JSON array/object if no record found
* A bool result/success flag in combination with previous options for a better handling.

Also, some internet providers may intercept your requests and return you a 404 http code. This does not means that your data are not found, but it's something wrong at transport level.

From [Wiki](https://en.wikipedia.org/wiki/HTTP_404#Soft_404_Errors):

```
In July 2004, the UK telecom provider BT Group deployed the Cleanfeed content blocking system, which returns a 404 error to any request for content identified as potentially illegal by the Internet Watch Foundation.
Other ISPs return a HTTP 403 "forbidden" error in the same circumstances. The practice of employing fake 404 errors as a means to conceal censorship has also been reported in Thailand and Tunisia.
In Tunisia, where censorship was severe before the 2011 revolution, people became aware of the nature of the fake 404 errors and created an imaginary character named "Ammar 404" who represents "the invisible censor".
```

Why not simply answer with something like this?

``` json
{
  "result": false,
  "error": {"code": 102, "message": "Validation failed: Wrong NAME."}
}
```

相关评论：

```
Actually, using HTTP status codes for REST is even more confusing down the road:
1) you see 4xx in your developer's toolbox and you can't say by just glancing at it whether server returned some sensible value or failed to process your request at all and then
2) all your error/exception/catch handlers should check what server returned as a response (mostly they don't since you'd have to do it on every service call) and many times
3) you get the same payload (type) on both success and error path leading to complicated/duplicated code... Very confusing indeed.
```

## 支持用http的错误码的观点
### 题外话 Http Error Code 错误码选择导图

在StackOverFlow 的问题 REST API error return good practices, 有一条比较不错的回答:

<https://stackoverflow.com/a/34324179/4854570>

原文如下：

<http://racksburg.com/choosing-an-http-status-code/>

### 正文

观点是为了统一性，纯粹性，纯粹的RESTful api既然用url作为资源请求，返回404似乎很有道理。
这样似乎还能让大家更便于理解错误码。毕竟RESTful API某种程度上，让前后端“彻底”(虽然不可能达到)分离，
前后端之间的约定好资源访问的规则，当然对错误码的规则也标准化最好。

然而，我不太认同这样的观点，stackoverflow中有关于RESTful api返回错误码的很多问题，随便举一例：

<https://stackoverflow.com/questions/2380554/rest-mapping-application-errors-to-http-status-codes?noredirect=1&lq=1>

可以看到，即使对于同一个问题的返回错误码，不同的人回答的结果不同，
这种返回错误没办法做到完全明确。即使有规定，现实API的返回错误也很难用http的错误码完全覆盖。

另一个缺陷引用一段知乎上找的网友的话：

```
全部返回200好像是天朝国情，如果异常时返回其他状态可能会被运营商劫持到导航页
```

关于返回http错误码，其实我觉得主要有一个好处，例如，假设对于内部错误500，
无论是来自于 Apache、Nginx的错误，还是后端设计的api内部错误，或者http server的错误，
都希望回复User ：“系统正在升级中，请耐心等待。。。”
这样，用http 500将减少判断条件。

## 我的思考
我个人认为有个更好的处理方式

为了对错误有明确的判断，我支持应用层api错误时，统一返回http 200，用JSON返回错误码及错误描述。

而错误码的设计，可以借用http错误码+三位api自定义错误码，
这样融合了两者的好处，如果熟悉http错误码，很容易猜到意思，而后三位错误码能判断错误具体原因。

例如，

1. 资源未找到，错误码统一成 404XXX的格式，用户未找到 错误码 404001， 电话未找到 404002等，
2. 参数错误统一成 400XXX格式，如用户账户格式错误 400001，电话号码格式错误 400002
3. 内部错误统一 500XXX格式，这样如果需要对内部错误统一处理，仅需判断错误码是不是500开头。
