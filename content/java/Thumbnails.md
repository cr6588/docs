---
title: "Thumbnails"
date: 2019-01-28T10:34:56+08:00
draft: true
---

后台用Thumbnails压缩时文件名的后缀会对图片大小产生影响，jpg格式改成png后缀后，原200多234007byte图片变成1.48Mb,使用jpg时116KB
Thumbnails.of(file.getInputStream()).scale(scale).toFile(outFile);