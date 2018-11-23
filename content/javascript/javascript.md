---
title: "javascript"
date: 2017-04-23 21:27
---

# javascript新手掉坑指南

## Python后台如果传回HTML文本，那么HTML文本需要转义字符转义

```
"initialPreview": "<img src=\'/file/{0}\' class=\'file-preview-image\' style=\'width:100%\'>".format(fid)
"initialPreview": "<img src=\'/file/%s\' class=\'file-preview-image\' style=\'width:100%%\'>"%(fid)
```
## JQuery element.show()不起作用，和css有关
[jQuery show() for Twitter Bootstrap css class hidden](http://stackoverflow.com/questions/14610412/jquery-show-for-twitter-bootstrap-css-class-hidden)

## JQuery 表单提交后监听事件处理
[Sending multipart/formdata with jQuery.ajax](http://stackoverflow.com/questions/5392344/sending-multipart-formdata-with-jquery-ajax)
```javascript
首先对表单元素进行submit监听，回调函数中使用$.ajax自己传输数据，
但是对要传输的数据使用new FromData(this)
$(document).on('submit', "form#data", function(e) {
        e.preventDefault();
        $.ajax({
            url: $(this).attr('action'),
            type: 'POST',
            data: new FormData(this),
            processData: false,
            contentType: false
        }).done(function(data) {
            console.log(data.templates);
            $("#filtered-image").empty().append(data.templates);
        });

});
```

## JQuery 动态增删改除元素或内容
当select内容本身为空时，直接调用add不起作用，应该用append，因为不一定支持add方法，w3school是最权威的，里面并没有add方法，但是有的jQuery文档可能有
```
empty() will remove all the contents of the selection.
remove() will remove the selection and its contents.
```
## JQuery “Uncaught SyntaxError: Unexpected token o”
[i-keep-getting-uncaught-syntaxerror-unexpected-token-o](http://stackoverflow.com/questions/8081701/i-keep-getting-uncaught-syntaxerror-unexpected-token-o)
```
 jQuery takes a guess about the datatype. It does the JSON parsing even though you're not calling getJSON()
Basically if the response header is text/html you need to parse, and if the response header is application/json it is already parsed for you.
```
## vue.js 访问model data的方式
```
var vm = new Vue({
  data:{
  a:1
  }
})
// `vm.a` 是响应的
vm.b = 2
```