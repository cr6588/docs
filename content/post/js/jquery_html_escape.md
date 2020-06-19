---
title: "复制按钮引发的问题"
date: 2020-06-04T11:10:00+08:00
draft: false
---

#### 简述

在做一个复制按钮时，初步以为只是复制以前的代码，但由于页面是大量的国际化code与翻译引发了一些转义问题。首页页面内容html源码是

````
<button class="h30-btn default-btn copyMsg">复制</button>
<div class="i18n-content">
&nbsp;&nbsp;&nbsp;&nbsp;"message.token.repeat.invalid": "Your request is being processed, please do not repeat!",<br>
&nbsp;&nbsp;&nbsp;&nbsp;"message.permission.validate.fail": "You do not have access to this page. If you have any queries, please contact your administrator.",
    ...
</div>
<script type="text/javascript">
    $(".copyMsg").click( function() {
        var msg = $(".i18n-content").text();
        var input = document.createElement('input');
        input.setAttribute('readonly', 'readonly');
        input.setAttribute('value', msg);
        document.body.appendChild(input);
        input.setSelectionRange(0, 9999);
        input.select();
        if(document.execCommand('copy')) {
            layer.tips('复制成功', this, {
                tips : [ 3, '#000' ],
                time : 400
            });
        }
        document.body.removeChild(input);
    });
    </script>
````
复制到粘贴板后会发现没有换行,因为input是不支持换行的。
1.于是将input的类型改成textarea发现复制到粘贴板后就只有textarea，完全没有div的内容。应该是值没有设置上去。通过input.value = msg;设置值而不是input.setAttribute('value', msg);

````
    var input = document.createElement('textarea');
    input.setAttribute('readonly', 'readonly');
    input.value = msg;
````

2.点击复制粘贴后仍然没有换行，改用html()。再次点击复制粘贴后还是没有换行

````
    var msg = $(".i18n-content").html();
````
3.复制粘贴后的内容&nbsp;&nbsp;与<br>这种空格显示与换行，以及大于小于，想着去掉，同时页面显示换行与空格，于是用正则替换同时在页面显示内容的div加入style="white-space: pre",内容与div要在同一行不能换行，不然显示会多出空格

````
    var msg = $(".i18n-content").html().trim().replace(/<br>/g, '\n').replace(/&nbsp;/g, ' ').replace(/&lt/g, '<').replace(/&gt/g, '>');
    ...

    <div class="i18n-content" style="white-space: pre">&nbsp;&nbsp;&nbsp;&nbsp;"message.token.repeat.invalid": "Your request is being processed, please do not
        ....
    </div>
````
4.粘贴后的内容依然后写特殊符号与显示的不符合，即html()返回的内容会将&转成&amp;等等，具体参考https://www.w3school.com.cn/html/html_entities.asp。之后还是觉得用text()方法得到内容。但text()是不会将&nbsp;与<br>改成换行的。于是在后台代码拼接内容的时候将<br>改成\n,&nbsp;直接使用 (空格)替代，加上div已有style="white-space: pre"样式会将空格与\n完整显示

````
    var msg = $(".i18n-content").text();
    ...
    #后台
    sb.append("\n");
    sb.append("    ");
    ...
````

5.运行一段时间后发现如果内容里若含有html标签如<a href="xxxxx" target="_blank">GO BUY</a>，显示在页面会将其渲染a标签，而不是源码，所以复制粘贴的时候就会只有GO BUY，而缺少<a href="xxxxx" target="_blank">与</a>，而我们实际是都需要的。 最后将div直接改成xmp标签。都说xmp标签已过时，用code与pre标签然后将<与>进行转义，但内容其实有可能也需要<或>号，故还是用xmp标签

````
    <xmp class="i18n-content">${i18ns}</xmp>
````
