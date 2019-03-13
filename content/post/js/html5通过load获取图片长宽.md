---
title: "html5通过load获取图片长宽"
date: 2018-10-31T17:48:43+08:00
categories: ["js"]
---


html5有naturalWidth，naturalHeight属性获取图片的长宽，但是必须在图片加载完成之后才能获取。可以使用load事件。但load在已有缓存时是不会执行的

    function setWidthHeight($span, nWidth, nHeight, imgIndex) {
        $span.text(nWidth + "X" + nHeight);
        if(imgIndex == 0) {
            $span.append("<em class=\"c-pink ml10 main-pic\">主图</em>");
        }
    }
    $(".small-img").each(function () {
        var imgDom = $(this)[0];
        nWidth = imgDom.naturalWidth;
        if(nWidth == 0) {
            $(this).load(function () {
                setWidthHeight($(this).parent().next("span"), imgDom.naturalWidth, imgDom.naturalHeight, $(".small-img").index(this));
            });
        } else {
            setWidthHeight($(this).parent().next("span"), imgDom.naturalWidth, imgDom.naturalHeight, $(".small-img").index(this));
        }
    });
