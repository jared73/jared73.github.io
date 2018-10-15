
---
title: 常见的java服务器日志分析
date: 2018-10-12 16:40:52
tags: 日志
categories: 服务器
---


>   有时候在工作中，我们需要对日志进行处理，这里我对我碰到的一些进行总结

* 提取数据
   首先可以打开日志，查看下结构
    ![image.png](https://upload-images.jianshu.io/upload_images/10298126-44e1ebf2184a9812.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  这里可以看到日志数据
  ![image.png](https://upload-images.jianshu.io/upload_images/10298126-f501e0997c20ecc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  &emsp;我们一般需要拿到访问某个接口的uid，time，app版本，登录手机的平台等信息
  首先我们先定位接口的数据：
&emsp;  格式这样：grep  查询的关键字  查询文件

      grep getArticleInfo ysz-gateway-2018_06_07-1.log

   ![image.png](https://upload-images.jianshu.io/upload_images/10298126-7a5ebfc36da2120b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  &emsp;如图，我们拿到了所有这个接口的相关数据，
  &emsp;我们这里需要提取这里的数据比如，我们要uid，这里我们需要管道处理
  &emsp; 解释下管道：我的理解大意是前边的结果作为参数传给后边处理，实际用多了，你们就体会到了，

      grep getArticleInfo ysz-gateway-2018_06_07-1.log | grep uid

 &emsp;获取到这个接口包含uid的行
![image.png](https://upload-images.jianshu.io/upload_images/10298126-d52a9c56774acb4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 &emsp;如下图，我们要提取[]里的内容，这里处理方法很多，我一般用awk
 ![image.png](https://upload-images.jianshu.io/upload_images/10298126-2ab7110824debe4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


    grep getArticleInfo ysz-gateway-2018_06_07-1.log | grep uid|awk -F "data:" '{print $2}'

这里awk -F是截取，这里是以data:为分割，输出第二段内容
![image.png](https://upload-images.jianshu.io/upload_images/10298126-6fae828d5feb045e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
和一般的json还差点啥，应该是去掉[]，我们这里还是用awk -F解决

    grep getArticleInfo ysz-gateway-2018_06_07-1.log | grep uid|awk -F "data:" '{print $2}'|awk -F "【" '{print $2}'|awk -F "】" '{print $1}'

然后就推荐使用jq插件了，yum install jq就可以安装了，
我们使用方法，比如要拿到uid，就这么写
继续管道后续

     grep getArticleInfo ysz-gateway-2018_06_07-1.log | grep uid|awk -F "data:" '{print $2}'|awk -F "【" '{print $2}'|awk -F "】" '{print $1}'|jq '.uid'

![image.png](https://upload-images.jianshu.io/upload_images/10298126-03311b127a8d8dfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

管道继续，有时候我们要去重，我们可以对uid排序，去重

     grep getArticleInfo ysz-gateway-2018_06_07-1.log | grep uid|awk -F "data:" '{print $2}'|awk -F "【" '{print $2}'|awk -F "】" '{print $1}'|jq '.uid'|wc -l  先看下行数
![image.png](https://upload-images.jianshu.io/upload_images/10298126-43106057bc352a7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    grep getArticleInfo ysz-gateway-2018_06_07-1.log | grep uid|awk -F "data:" '{print $2}'|awk -F "【" '{print $2}'|awk -F "】" '{print $1}'|jq '.uid'|sort|uniq|wc -l

![image.png](https://upload-images.jianshu.io/upload_images/10298126-f8c835f730877326.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到去重以后由145变成了92，
现在就拿到数据了，至于后边数据分析，我们后边会讲到
