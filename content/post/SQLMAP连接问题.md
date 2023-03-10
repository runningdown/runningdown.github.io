---
title: SQLMAP连接问题
description: 某次遇到sqlmap无法连接target的问题，但是使用chrome可以正常访问。
date: 2023-03-02
image: https://typora-mine.oss-cn-beijing.aliyuncs.com/typora/image-20230302195439003.png
categories:
    - SQLAMP
    - 奇怪的知识
---





## 0x01 背景

某次遇到sqlmap无法连接target的问题，但是使用chrome可以正常访问，关于报错信息大部分如下图，意思是无法与target连接，但是使用浏览器可以正常访问target，甚至说在burp里使用payload也可以正常执行。

![image-20230302195439003](https://typora-mine.oss-cn-beijing.aliyuncs.com/typora/image-20230302195439003.png)



## 0x02 开始

刚开始遇到无法连接目标的问题的时候。还以为是请求过高导致宕机，或者是被banip，所以降低请求频率、更换代理节点等等措施都使用过，但是依旧无效。

于是重新翻请求，发现存在如下图所示信息

![image-20230302200407588](https://typora-mine.oss-cn-beijing.aliyuncs.com/typora/image-20230302200407588.png)

意为我在POST中对参数进行了标记，sqlmap将对这个参数进行测试。但是问题来了，在POST请求中我并没有进行标记，如下图所示，仔细观察发现在header中存在两个星号

![image-20230302200816820](https://typora-mine.oss-cn-beijing.aliyuncs.com/typora/image-20230302200816820.png)

也就是说sqlmap会把header中的星号当作POST参数进行注入（因为有设置对header的测试，所以刚开始并没有觉得有什么不对），接下来在sqlmap中使用 **-v 5** 将日志信息调到最详细，可以看到的确如此，从 **Accept** 开始就被sqlmap当作POST数据包进行测试。

![image-20230302201057195](https://typora-mine.oss-cn-beijing.aliyuncs.com/typora/image-20230302201057195.png)

## 0x03 解决

那么开始解决这个问题，首先这里说结论，是**Content-Length**的锅，删除即可。

最开始以为是Windows换行符的问题，于是将请求包从Windows复制到Linux重新测试，但是依旧无法解决。

之后以为是SQLMAP的问题，更换最新版本或者切换旧版本依旧无法解决。

最后依次删除header进行测试，定位到**Content-Length**，删掉让SQLMAP自动补全一个即可。

## 0x04后续

这个问题特别有意思，在此记录一下，另外测试如果把**Content-Length**放到header的最后面可不可以解决问题呢？测试一下，发现SQLMAP重新在POST参数前加上了 **%0A%0A** ，所以还是重新生成或者加大数值靠谱一点。

![image-20230302202408680](https://typora-mine.oss-cn-beijing.aliyuncs.com/typora/image-20230302202408680.png)

