---
title: "networks-interview"
date: 2017-06-05 20:31
---

# HTTP
## HTTP POST数据传输的四种方式
请求头中 Content-Type 的四种方式:
### application/x-www-form-urlencoded
最常见的普通的表单传输, 在原生 `<form>` 表单中, 如果不设置 `enctype` 属性, 最终就是  application/x-www-form-urlencoded 方式提交数据.

### multipart/form-data
该方式一般是传文件用的.

### application/json
传递 json 数据用. 现在的 Restful API 很多都使用 json 直接传输数据, 还有 SinglePage WebAPP

### text/xml
数据传输的另一种 xml. XML-RPC

## HTTP 传输大文件大图片的方法和优化思路

## cankao
- [HTTP-请求、响应、缓存](https://cnbin.github.io/blog/2016/02/20/http-qing-qiu-,-xiang-ying-,-huan-cun/)
- [HTTP协议知识扫盲](http://movesan.me/2017/03/06/http/)
- [四种常见的 POST 提交数据方式](https://imququ.com/post/four-ways-to-post-data-in-http.html)