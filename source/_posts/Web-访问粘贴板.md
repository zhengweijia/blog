---
title: Web 访问粘贴板
date: 2018-10-28 03:24:48
tags:
---

### 浏览器 Web 访问剪切板图片

##### 前言

有时候，我们希望能访问用户的剪切板，来实现一些方便用户的功能；但是另一方面，剪切板里的数据对用户来说又是非常隐私的，所以浏览器在获取信息方面有安全限制，同时也提供访问接口。

前段时间由于业务功能，需要实现当用户在富文本里进行粘贴操作的时候，如果用户复制的是图片，需要将图片上传服务器后，插入到文本内；看似合情合理的要求，却有很多坑

##### 一、什么时候能访问剪切板

1、在用户触发 onPaste 事件时（只能是用户进行事件触发），通过事件对象（event）获取。[官方文档](https://developer.mozilla.org/en-US/docs/Web/Events/paste) 

```javascript
let text = (event.clipboardData || window.clipboardData).getData('text');
```

2、Chrome 新增异步剪贴板 API，可以跟用户申请访问权限，得到允许后则能访问。参考知乎的一篇[文章](https://zhuanlan.zhihu.com/p/34698155)

##### 二、访问剪切板里的图片

###### 先说结论

| 从文件系统复制图片 | mac os能拿到最后一张；windows 拿不到数据 |
| ------------------ | ---------------------------------------- |
| 截图               | mac 和 windows 能获取                    |
| 在浏览器界面复制   | mac 和 windows 能获取                    |
|                    |                                          |

windows 为什么一张都拿不到呢，用户在文件系统复制的文件，在剪切板里存的并不是图片（猜测存的是图片的标记），所以是拿不到。



###### 代码如下

```javascript
/**
*	触发 paste 事件，响应方法
*/
function paste(event) {
    // 拿到数据
    let items = (event.clipboardData || event.originalEvent.clipboardData).items;
	let imgList = []; // 获得图片数据，可以直接写入到 <img src=''> 的src内进行展示 
    let strList = [];
    for (let item of items) {
      // 如果拿到的数据是文件
      if (item.kind === 'file') {
        let blob = item.getAsFile();
        let reader = new FileReader();
        blobList.push(blob);
        reader.onload = ()=> {
          imgList.push(reader.result);
        }
        reader.readAsDataURL(blob);
      } else if(item.kind === 'string') {
        // 如果拿到的数据是字符串
        item.getAsString((s)=>{
          strList.push(s);
        });
      }
    }
}
```



最后，想要完全实现用户 Contr+C、Contr+V 来发帖，目前还做不到。有一种替代方案，将图片拖动到富文本区域，这个操作也是很方便的，浏览器也都支持。



另外有一个公司的商业产品，他家开发的浏览器插件能支持在桌面复制图片后，粘贴到浏览器里。可惜目前只支持 windows 系统，mac os 还没有提供；

他们的方案我大致猜测是：安装浏览器插件并且允许运行后，当你在浏览器触发paste粘贴事件后，js调用插件方法，插件去访问系统里的文件（插件其实也是windows 的一个软件，所以能访问文件系统），然后返回给页面的 js。