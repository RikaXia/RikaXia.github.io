---
layout:     post
title:      create-react-app+fileSaver+Blob对象创建指定文件并下载
subtitle:   将文件导出保存到本地
date:       2018-11-12-
author:     Rika
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - React
---   
 
# 需求
需要将文件导出保存到本地
# 效果
![tab2.gif](https://upload-images.jianshu.io/upload_images/6949865-b91e61904c624cce.gif?imageMogr2/auto-orient/strip)

# 什么是Blob
参考文章如下 ：    

[前端利用Blob对象创建指定文件并下载](https://segmentfault.com/a/1190000015026760)    

[MDN的blob概念](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)  

Blob对象表示一个不可变、原始数据的类文件对象。Blob 表示的不一定是JavaScript原生格式的数据。File接口基于Blob，继承了blob的功能并将其扩展使其支持用户系统上的文件。

## 构造函数  
```js
var aBlob = new Blob( array, options );
```
- array 是一个由ArrayBuffer（二进制数据缓冲区）、ArrayBufferView（二进制数据缓冲区的array-like视图）、Blob、DOMString等对象构成的Array，或者其他类似对象的混合体，它将会被放进Blob。DOMStrings会被编码为UTF-8。
- options 是可选的，它可能会指定如下两个属性：
    - type，默认值为 ""，它代表了将会被放入到blob中的数组内容的MIME类型。
    - endings，默认值为"transparent"，用于指定包含行结束符n的字符串如何被写入。 它是以下两个值中的一个： "native"，代表行结束符会被更改为适合宿主操作系统文件系统的换行符，或者 "transparent"，代表会保持blob中保存的结束符不变。

## 示例

```js
var debug = {hello: "world"};
var blob = new Blob([JSON.stringify(debug, null, 2)], {type : 'application/json'});
```

# 应用
安装fileSaver[(github地址)](https://github.com/eligrey/FileSaver.js)  

```js
npm install file-saver --save
```

```js
// 引入模块
import FileSaver from 'file-saver';  

export default class App extends Component { 
  
  // 点击下载
  downFile = () => {
    this.download();
  }

  // 下载文件
  download() {
    // 要输出的文件内容
    let particleData = store.getEmitterJson;
    
    // 将指定数据转换为 JSON 字符串
    let content = JSON.stringify(particleData, null, 2);
    
    // 数组内容的MIME类型为json
    let type = "data:application/json;charset=utf-8";
    
    // 实例化Blob对象，并传入参数        
    let blob =new Blob([content], {type: type});

    let isFileSaverSupported = false;
    try {
      isFileSaverSupported = !!new Blob();
    } catch (e) {
      console.log(e);
    }

    if (isFileSaverSupported) {
      FileSaver.saveAs(blob,"particleData.json");
    }else{
      FileSaver.open(encodeURI(type + "," + content));
    }
  } 
}
  
```


