---
layout: post_layout
title: 抓包学习RTSP协议
category: [live media]
---

### 1.关于RTSP
&ensp;&ensp; 最近需要学习关于流媒体的一些知识，需要了解RTSP,RTP,RTCP这些协议，我认为要学习这些协议的一个方法就是抓取具体的相关协议的报文，然后分析这些报文。关于rtsp协议的作用和介绍
我在这里就不多做讲解，可以查看[RTP/RTCP/RTSP协议详解](http://taishanmayi.github.io/live%20media/2016/03/24/rtsp.html)。在本文中，我主要是分析抓取的报文来分析rtsp协议。  

### 2.使用Wireshark抓取RTSP报文
&ensp;&ensp; 我在网络上找了一个rtsp的测试源，地址是rtsp://218.204.223.237:554/live/1/66251FC11353191F/e7ooqwcfbqjoo80j.sdp。然后打开Wireshark，
设置wireshark的捕捉过滤器为`src or dst host 218.204.223.237`,开启捕获，用VLC打开前面rtsp的地址，将wireshark捕捉到的报文保存到文件。抓取到的结果如下： 

![rtsp_capture](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_capture.png)

&ensp;&ensp; 在如上报文可以看出，在本机和218.204.223.237建立了TCP连接后，马上发送了OPTIONS,DESCRIBE,SETUP,PLAY几个请求到rtsp服务器，之后数据流开始传送。

### 3.OPTIONS消息
&ensp;&ensp; OPTIONS消息由client发出，用于向server请求server支持的方法，请求到这些方法后，client只能向server发送server支持的方法请求。

![options_client](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_option.png)

&ensp;&ensp; client端发送的消息包括：

+ 1.request 方法：OPTION，地址是rtsp://218.204.223.237:554/live/1/66251FC11353191F/e7ooqwcfbqjoo80j.sdp。
+ 2.Cseq = 2   是标示发送该请求的序号，在client收到了来自server端的回复后，可以根据收到的Cseq来判断是针对哪一个请求的回复。
+ 3.User-Agent 代理的名称，这里填任何值都是可以的，没有具体的作用。  

&ensp;&ensp; server端到clinet端的针对OPTION方法的回复如下图： 

![options_server](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_OPTIONS_server.png)

&ensp;&ensp;其中：

+ 1.Response 状态码：标示针对发送的请求的状态,状态码的含义如下。  

```
         状态码  =     "100"      ; 继续
                |     "200"      ; OK
                |     "201"      ; 已创建
                |     "250"      ; 存储空间不足
                |     "300"      ; 有多个选项
                |     "301"      ; 被永久移除
                |     "302"      ; 被临时移除
                |     "303"      ; 见其他
                |     "304"      ; 没有修改
                |     "305"      ; 使用代理
                |     "400"      ; 错误的请求
                |     "401"      ; 未通过认证
                |     "402"      ; 需要付费
                |     "403"      ; 禁止
                |     "404"      ; 没有找到
                |     "405"      ; 不允许该方法
                |     "406"      ; 不接受
                |     "407"      ; 代理需要认证
                |     "408"      ; 请求超时
                |     "410"      ; 不在服务器
                |     "411"      ; 需要长度
                |     "412"      ; 预处理失败
                |     "413"      ; 请求实体过长
                |     "414"      ; 请求-URI过长
                |     "415"      ; 媒体类型不支持
                |     "451"      ; 不理解此参数
                |     "452"      ; 找不到会议
                |     "453"      ; 带宽不足
                |     "454"      ; 找不到会话
                |     "455"      ; 此状态下此方法无效
                |     "456"      ; 此头部域对该资源无效
                |     "457"      ; 无效范围
                |     "458"      ; 参数是只读的
                |     "459"      ; 不允许合控制
                |     "460"      ; 只允许合控制
                |     "461"      ; 传输方式不支持
                |     "462"      ; 无法到达目的地址
                |     "500"      ; 服务器内部错误
                |     "501"      ; 未实现
                |     "502"      ; 网关错误
                |     "503"      ; 无法得到服务
                |     "504"      ; 网关超时
                |     "505"      ; 不支持此RTSP版本
                |     "551"      ; 不支持选项
                |     扩展码
```
+ 2.public 标示了server能够解释的方法,在本次测试的源中，可以解释DESCRIBE, SETUP, TEARDOWN, PLAY, PAUSE, OPTIONS, ANNOUNCE, RECORD这几种方法。
+ 3.Cseq = 2 表示是针对前一条OPTIONS请求的回复。

&ensp;&ensp; 由此可以看出，OPTIONS方法的作用就是为了获取到server支持哪几种方法。

### 4.DESCRIBE消息
&ensp;&ensp; client在发送OPTIONS请求后，发送了DESCRUBE请求。DESCRIBE方法的作用是：请求URI所指定的媒体描述信息，一般是SDP信息。clinet
通过Accept头指定客户端可以接受的媒体描述信息类型。client发送给server的DESCRIBE方法报文格式如下：

![describe_client](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_describe_client.png)

&ensp;&ensp; 发送的报文很简单，指定需要请求的方法和请求的地址就OK了。

+ 1.DESCRIBE rtsp://218.204.223.237:554/live/1/66251FC11353191F/e7ooqwcfbqjoo80j.sdp RTSP/1.0\r\n  指定请求的方法和地址
+ 2.Accept: application/sdp\r\n 指定能够接收的媒体描述信息类型。 媒体描述信息的类型有application/sdp, application/rtsl,application/mheg这几种。

&ensp;&ensp; server回复的消息比较复杂：
![describe_server](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_describe_server_2.png)

&ensp;&ensp; 在该回复的消息内容中关键的信息有：

+ 1.Content type : 回复的描述信息的类型。一般是application/sdp。
+ 2.Session Description Protocol中的信息是sdp的具体内容。 

    ![sdp](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_dsp.png)

    Session Description Protocol Version (v): 0  ; 协议版本
    Owner/Creator, Session Id (o): - 1 1 IN IP4 0.0.0.0 ； 包含与会话所有者有关的参数
    Connection Information (c): IN IP4 218.204.223.237 ； 多媒体会话而建立的连接的信息，包含了媒体流使用的IP地址
    Time Description, active time (t): 0 0 ；表示会话的开始时间和结束时间
    Media Description, name and address (m): video 0 RTP/AVP 96 ； 表示server上的媒体信息，此处表示为vedio，支持的传输协议为RTP/AVP
    Bandwidth Information (b): AS:120 ； 表示带宽信息，此处表示带宽为120Kb/s
    其他的是一些回话属性的描述信息。如`Media Attribute (a): framerate:10`,`Media Attribute (a): framesize:96 352-288`。

关于SDP的详细信息可以参考[SDP协议](http://blog.csdn.net/haidonglin/article/details/8727933)。

### 5.SETUP方法
&ensp;&ensp; SETUP方法用于设置一些参数，通过Transport头字段列出可接受的传输选项，建立会话。

![setup_client](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_setup.png)

&ensp;&ensp; 客户端发送的消息:

+ 1.Request: SETUP rtsp://218.204.223.237:554/live/1/66251FC11353191F/e7ooqwcfbqjoo80j.sdp/trackID=1 RTSP/1.0\r\n  包含了请求的地址 trackID=1表示要设置的通道。
+ 2.Transport: RTP/AVP;unicast;client_port=53250-53251  表示设置的传输协议，传输方式，端口号

服务器的回复：

![setup_server](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_setup_server.png)

服务器的回复中比较简单，表示接受SETUP消息OK,通过Transport头字段返回选择的具体传输选项，并返回建立的Session ID,此处返回的Session ID为137499083027321。

### 6.PLAY方法
&ensp;&ensp; 使用SETUP方法建立连接之后，client端发送PLAY方法到server端，启动播放。
&ensp;&ensp; client发送给server端只需要指定请求方法为PLAY，请求的地址为rtsp://218.204.223.237:554/live/1/66251FC11353191F/e7ooqwcfbqjoo80j.sdp/即可。
&ensp;&ensp; server端的回复：
```
   RTP-Info: url=rtsp://218.204.223.237:554/live/1/66251FC11353191F/e7ooqwcfbqjoo80j.sdp/trackID=1;seq=0;rtptime=0\r\n
```
表示了请求的播放源的信息。

### 7.TEARDOWN方法
&ensp;&ensp; 在启动数据传输之后，client可以使用TEARDOWN方法终止传输。
&ensp;&ensp; 客户端发送：

![teardown](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp_teardown_client.png)

客户端发送TEARDOWN请求到相应地址，并发送要关闭的seesion id到server端，之后断开数据流传输。在我抓取的报文中，没有抓取到server端针对TEARDOWN请求的回复，而在看其他的同学抓取的报文
中，都有针对TEARDOWN请求的回复消息，返回OK，connection状态，和成功关闭的seesion id。

### 8.总结
&ensp;&ensp; 本文抓取了一个rtsp视频流的运行过程的报文，只分析了其中的RTSP报文相关的几个方法，以便了解RTSP视频流传输的大概流程，而在RTSP视频传输过程中更为复杂的RTP和RTCP协议的通信
过程，在后续有时间的时候会再做分析。
&ensp;&ensp; 关于RTSP协议通信的过程，有同学绘制了一张流程图，放在此处，有助理解。

![rtsp](https://github.com/taishanmayi/taishanmayi.github.io/raw/master/assets/img/rtsp/rtsp.jpg)