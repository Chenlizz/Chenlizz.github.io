---
title: 单点登录（SSO）
date: 2019-08-05 17:27:17
tags:
categories: 
- 单点登录
---

多个应用系统只需登录一次，可以访问其他相互信任的应用系统。

## 不同域下的单点登录（CAS流程）
 - 用户访问app，app是需要登录的，用户现在还没有登录。
 - 跳转到CAS Server，即SSO登录系统，弹出登录页。
 - 用户输入信息登录，将登陆状态写入SSO的Session，浏览器中写入SSO域下的Cookie。
 - SSO系统登录完成后会生成一个ST（Servies Ticket），然后跳转到app，同时ST作为参数一并传过去。
 - app拿到ST后，从后台向SSO发送请求，验证ST是否有效。
 - 验证通过后，app系统将登陆状态写入Session，并设置app域下的Cookie。
 至此，单点登录完成。

 - 用户访问app2系统，app2没有登录，跳转至SSO。
 - SSO已登录，无需重新认证。
 - SSO生成ST，并随着跳转至app2将ST作为参数传递出去。
 - app2拿到ST，后台访问SSO，验证ST是否有效。
 - 验证成功后，app2将登陆状态写入Session，并在app2域下写入Cookie。