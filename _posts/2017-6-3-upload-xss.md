---
title: 翻译 upload-xss
date: 2017-6-3 1:50:30
tags: Web安全
toc: true
---

> 没有完全按原文翻译
> 原文 https://brutelogic.com.br/blog/file-upload-xss/?utm_source=ReviveOldPost&utm_medium=social&utm_campaign=ReviveOldPost

上传文件是XSS的绝佳机会。带有上传个人资料图片的用户受限区域无处不在，为寻找开发者的错误这里提供了更多机会。如果它恰好是一个 self XSS，那就看看前一篇文章 http://brutelogic.com.br/blog/leveraging-self-xss/

基本上有如下攻击点：

### Filename
---
文件名本身他可能输出到页面，
![](https://i1.wp.com/brutelogic.com.br/blog/wp-content/uploads/2016/04/xss-gif-filename.gif)

尽管不被注意，可以在这里练习 [w3school](http://www.w3schools.com/jsref/tryit.asp?filename=tryjsref_fileupload_value)

### Metadata
---
使用 [exiftool](http://www.sno.phy.queensu.ca/~phil/exiftool/) 修改 EXIF metadata ,导致 xss
> $ exiftool -FIELD=XSS FILE

Example:
```
$ exiftool -Artist=’ “><img src=1 onerror=alert(document.domain)>’ brute.jpeg
```
![](https://i2.wp.com/brutelogic.com.br/blog/wp-content/uploads/2016/04/exif-brute-collage.jpg)

### Content
---
如果应用允许上传 SVG 扩展文件，文件内容如下也会触发 xss

> <svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)"/>

一个例子在[brutelogic.com.br/poc.svg](http://brutelogic.com.br/poc.svg) 查看源码可看 poc

### Source
---
构建一个 GIF image 携带 js payload，去 bypass CSP , 前提是在同域下 ：
![](https://i2.wp.com/brutelogic.com.br/blog/wp-content/uploads/2016/04/xss-gif-source.gif)
去创建一个 image 使用如下内容和 .gif 后缀
> GIF89a/*<svg/onload=alert(1)>*/=alert(document.domain)//;

开头代表 GIF 文件(GIF89a)，使用变量赋值,为了防止图像识别为text/HTML MIME类型，注释XSS攻击向量，从而允许通过请求文件执行 payload。
正如下面看到的，如果用 PHP 函数 exif_imagetype() and getimagesize() 识别，会识别为 GIF 文件 
![](https://i2.wp.com/brutelogic.com.br/blog/wp-content/uploads/2016/04/xss-gif.png)
For more file types that can have its signature as ASCII characters used for a javascript variable assignment, check [this](https://en.wikipedia.org/wiki/List_of_file_signatures).

绕过 GD library 的例子 [here](https://github.com/d0lph1n98/Defeating-PHP-GD-imagecreatefromgif)



