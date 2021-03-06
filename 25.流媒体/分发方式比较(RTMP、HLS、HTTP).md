## 分发方式比较(RTMP、HLS、HTTP)

互联网上的两种主要的分发方式：HLS和RTMP，什么时候用谁，完全决定于应用场景        
&emsp;&emsp;还有其他的分发方式，这些分发方式不属于互联网常见和通用的方式，不予以比较：

- UDP：比如YY的实时应用，视频会议等等，或者RTSP之类。这类应用的特点就是实时性要求特别高，以毫秒计算。TCP家族协议根本就满足不了要求，所以HTTP/TCP都不靠谱。这类应用没有通用的方案，必须自己实现分发（服务端）和播放（客户端）
- P2P：比如RTMFP或者各家自己的协议。这类应用的特点是节省带宽。目前PC/flash上的RTMFP比较成熟，Android上的P2P属于起步群雄纷争标准不一，IOS上P2P应该没有听说过
- RTSP：这种不是互联网上的主要应用，在其他领域比如安防等有广泛应用



### 1、HTTP
HTTP说的是HTTP流，比如各大视频网站的点播流，HTTP本质上还是文件分发
#### 1.1 HTTP优势：
- 性能很高：HTTP的性能没得说，协议简单，各种HTTP高性能服务器也完善。如果分发的量特别大，比如点播视频网站，没有直播的实时性要求，HTTP协议是最好选择
- 没有碎片：HTTP比HLS没有碎片，HTTP分发大文件会比小文件分发方便很多。特别是存储，小文件的性能超低，是个硬伤
- 穿墙：互联网不可能不开放HTTP协议，否则就不叫互联网。所以任何端口封掉，也不会导致HTTP流看不了。（不过RTMP也能穿墙，用RTMPT协议）

#### 1.2 HTTP的劣势：
- 实时性差：基本上没有实时性这个说法。
- 原生支持不好：就PC上flash对于HTTP流支持还可以，Android/IOS上似乎只能mp4，总之移动端对于HTTP的支持不是很完善

#### 1.3 HTTP的也分为几种：
- HTTP progressive：早期流媒体服务器分发http文件时，以普通的http文件分发，这种叫做渐进式下载，意思就是如果文件很大比如1小时时长1GB大小，想从中间开始播放是不行的。但这种方式已经是作古了，很多http服务器支持http文件的seek，就是从中间开始播放
- HTTP stream：支持seek的HTTP流，比如各家视频网站的点播分发方式。或者稍微复杂点的，比如把一个大文件切几段之后分发。目前在pc/flash上点播国内的主流分发是这种方式
- HLS：这种是现在适配方式最广（除了flash, 需要额外的as库支持），在PC上有vlc，Android/IOS原生播放器就支持播放HLS，HTML5里面的url可以写HLS地址。总之，在移动端是以HLS为主
- HDS：adobe自己的HLS，垃圾
- DASH：各家提出的HLS，目前还没有广泛应用


### 2、HLS
#### 2.1 什么是HLS协议：    
&emsp;&emsp;简单讲就是把整个流分成一个个小的，基于HTTP的文件来下载，每次只下载一些，基于H5播放直播视频时引入的一个.m3u8的文件，这个文件就是基于HLS协议，存放视频流元数据的文件       
&emsp;&emsp;每一个 .m3u8 文件，分别对应若干个ts文件，这些ts文件才是真正存放视频的数据，m3u8文件只是存放了一些ts文件的配置信息和相关路径，当视频播放时，.m3u8是动态改变的，video标签会解析这个文件，并找到对应的ts 文件来播放，所以一般为了加快速度，.m3u8放在web服务器上，ts文件放在CDN上      
&emsp;&emsp;.m3u8文件，其实就是以UTF-8编码的m3u文件，这个文件本身不能播放，只是存放了播放信息的文本文件：
```ruby
#EXTM3U                 m3u文件头
#EXT-X-MEDIA-SEQUENCE   第一个TS分片的序列号
#EXT-X-TARGETDURATION   每个分片TS的最大的时长
#EXT-X-ALLOW-CACHE      是否允许cache
#EXT-X-ENDLIST          m3u8文件结束符
#EXTINF                 指定每个媒体段(ts)的持续时间（秒），仅对其后面的URI有效
livestream-536.ts
#EXTINF:10.004, no desc
livestream-537.ts
#EXTINF:10.023, no desc
livestream-538.ts
#EXTINF:10.007, no desc
livestream-539.ts
#EXTINF:10.003, no desc
livestream-540.ts
#EXTINF:10.000, no desc
livestream-541.ts
#EXTINF:9.475, no desc
livestream-542.ts
```
video 标签播放 hls 协议视频：
```ruby
 <video controls autoplay>  
     <source src="http://10.66.69.77:8080/hls/mystream.m3u8" type="application/vnd.apple.mpegurl" />  
     <p class="warning">Your browser does not support HTML5 video.</p>  
 </video> 
```
#### 2.2 HLS直播延时：
&emsp;&emsp;HLS协议是将直播流分成一段一段的小段视频去下载播放的，所以假设列表里面的包含5个ts文件，每个ts文件包含5秒的视频内容，那么整体的延迟就是25秒。因为当你看到这些视频时，主播已经将视频录制好上传上去了，所以这样会产生的延迟。当然可以缩短列表的长度和单个ts文件的大小来降低延迟，极致来说可以缩减列表长度为1，并且ts的时长为1s，但是这样会造成请求次数增加，增大服务器压力，当网速慢时回造成更多的缓冲，所以苹果官方推荐的ts时长时10s，所以这样就会大改有30s的延迟
#### 2.3 视频采集：
&emsp;&emsp;**视频编码**：所谓视频编码就是指通过特定的压缩技术，将某个视频格式的文件转换成另一种视频格式文件的方式，我们使用的iphone录制的视频，必须要经过编码，上传，解码，才能真正的在用户端的播放器里播放       
&emsp;&emsp;**编解码标准**：视频流传输中最为重要的编解码标准有国际电联的H.261、H.263、H.264，其中HLS协议支持H.264格式的编码。      
&emsp;&emsp;**音频编码**：同视频编码类似，将原始的音频流按照一定的标准进行编码，上传，解码，同时在播放器里播放，当然音频也有许多编码标准，例如PCM编码，WMA编码，AAC编码等等，HLS协议支持的音频编码方式是AAC编码

![](https://github.com/ZongYuWang/image/blob/master/%E6%B5%81%E5%AA%92%E4%BD%93/HLS2.png)

#### 2.4 HLS的主要优势：
- 性能高：和HTTP一样
- 穿墙：和HTTP一样
- 原生支持很好：IOS上支持完美。Android上支持差些。PC/flash上现在也有各种as插件支持HLS

#### 2.5 HLS的主要劣势：
- 实时性差：基本上HLS的延迟在10秒以上
- 文件碎片：若分发HLS，码流低，切片较小时，小文件分发不是很友好。特别是一些对存储比较敏感的情况，比如源站的存储，嵌入式的SD卡


### 3、RTMP
&emsp;&emsp;RTMP（Real Time Messaging Protocol）是Macromedia开发的一套视频直播协议，现在属于 Adobe。和HLS一样都可以应用于视频直播，区别是RTMP基于flash无法在ios的浏览器里播放，但是实时性比HLS要好。所以一般使用这种协议来上传视频流，也就是视频流推送到服务器

&emsp;&emsp;RTMP协议规定，播放一个流媒体有两个前提步骤：第一步，建立一个网络连接（NetConnection），第二步，建立一个网络流（NetStream）。其中，网络连接代表服务器端应用程序和客户端之间基础的连通关系。网络流代表了发送多媒体数据的通道。服务器和客户端之间只能建立一个网络连接，但是基于该连接可以创建很多网络流。他们的关系如图所示：

![](https://github.com/ZongYuWang/image/blob/master/%E6%B5%81%E5%AA%92%E4%BD%93/Audio-video-coding5.png)

&emsp;&emsp;播放一个RTMP协议的流媒体需要经过以下几个步骤：握手，建立连接，建立流，播放。RTMP连接都是以握手作为开始的，建立连接阶段用于建立客户端与服务器之间的“网络连接”；建立流阶段用于建立客户端与服务器之间的“网络流”，播放阶段用于传输视音频数据

##### 握手：
一个RTMP连接以握手开始，双方分别发送大小固定的三个数据块
- 握手开始于客户端发送C0、C1块。服务器收到C0或C1后发送S0和S1。
- 当客户端收齐S0和S1后，开始发送C2。当服务器收齐C0和C1后，开始发送S2。
- 当客户端和服务器分别收到S2和C2后，握手完成。      

![](https://github.com/ZongYuWang/image/blob/master/%E6%B5%81%E5%AA%92%E4%BD%93/Audio-RTMP2.png)

##### 建立网络连接：
- 客户端发送命令消息中的“连接”(connect)到服务器，请求与一个服务应用实例建立连接。
- 服务器接收到连接命令消息后，发送确认窗口大小(Window Acknowledgement Size)协议消息到客户端，同时连接到连接命令中提到的应用程序。
- 服务器发送设置带宽()协议消息到客户端。
- 客户端处理设置带宽协议消息后，发送确认窗口大小(Window Acknowledgement Size)协议消息到服务器端。
- 服务器发送用户控制消息中的“流开始”(Stream Begin)消息到客户端。
- 服务器发送命令消息中的“结果”(_result)，通知客户端连接的状态

![](https://github.com/ZongYuWang/image/blob/master/%E6%B5%81%E5%AA%92%E4%BD%93/Audio-RTMP3.png)

##### 建立网络流（NetStream）：
- 客户端发送命令消息中的“创建流”（createStream）命令到服务器端。
- 服务器端接收到“创建流”命令后，发送命令消息中的“结果”(_result)，通知客户端流的状态。       
    

![](https://github.com/ZongYuWang/image/blob/master/%E6%B5%81%E5%AA%92%E4%BD%93/Audio-RTMP4.png)   


##### 播放（Play）：
- 客户端发送命令消息中的“播放”（play）命令到服务器。
- 接收到播放命令后，服务器发送设置块大小（ChunkSize）协议消息。
- 服务器发送用户控制消息中的“streambegin”，告知客户端流ID。
- 播放命令成功的话，服务器发送命令消息中的“响应状态” NetStream.Play.Start & NetStream.Play.reset，告知客户端“播放”命令执行成功。
- 在此之后服务器发送客户端要播放的音频和视频数据。

![](https://github.com/ZongYuWang/image/blob/master/%E6%B5%81%E5%AA%92%E4%BD%93/Audio-RTMP5.png)

| - | 协议 | 原理 | 延时 | 优点 | 使用场景 | 
| - | - | - | - | - | - | 
| RTMP |长链接TCP| 每个时刻的数据收到后立刻发送 | 2S | 延时低 | 即时互动| 
| HLS | 短链接HTTP | 集合一段时间数据生成ts切片文件更新m3u8文件 | 10s-30s | 跨平台 | H5直播


#### 3.1  RTMP本质上是流协议，主要的优势如下：
- 实时性高：RTMP的实时性在3秒之内，经过多层CDN节点分发后，实时性也在3秒左右。在一些实时性有要求的应用中以RTMP为主
- 支持加密：RTMPE和RTMPS为加密协议。虽然HLS也有加密，但在PC平台上flash对RTMPE/RTMPS支持应该比较不错
- 稳定性高：在PC平台上flash播放的最稳定方式是RTMP，如果做CDN或者大中型集群分发，选择稳定性高的协议一定是必要的。HTTP也很稳定，但HTTP是在协议上稳定；稳定性不只是服务端的事情，在集群分发，服务器管理，主备切换，客户端的支持上，RTMP在PC分发这种方式上还是很有优势
- 编码器接入：编码器输出到互联网（还可以输出为udp组播之类广电应用），主要是RTMP。比如专业编码器，或者flash网页编码器，或者FMLE，或者ffmpeg，或者安防摄像头，都支持RTMP输出。若需要接入多种设备，比如提供云服务；或者希望网页直接采集摄像头；或者能在不同编码器之间切换，那么RTMP作为服务器的输入协议会是最好的选择
- 系统容错：容错有很多种级别，RTMP的集群实现时可以指定N上层，在错误时切换不会影响到下层或者客户端，另外RTMP的流没有标识，切到其他的服务器的流也可以继续播放。HLS的流热备切换没有这么容易。若对于直播的容错要求高，比如降低出问题的概率，选择RTMP会是很好的选择
- 可监控：在监控系统或者运维系统的角度看，流协议应该比较合适监控。HTTP的流监控感觉没有那么完善。这个不算绝对优势，但比较有利

#### 3.2 RTMP的劣势：
- 协议复杂：RTMP协议比起HTTP复杂很多，导致性能低下。测试发现两台服务器直连100Gbps网络中，HTTP能跑到60Gbps，但是RTMP只能跑到10Gbps，CPU占用率RTMP要高很多。复杂协议导致在研发，扩展，维护软件系统时都没有HTTP那么方便，所以HTTP服务器现在大行其道，apache/nginx/tomcat，N多HTTP服务器；而RTMP协议虽然早就公开，但是真正在大规模中分发表现良好的没有，adobe自己的FMS在CDN中都经常出问题
- Cache麻烦：流协议做缓存不方便。譬如点播，若做RTMP流协议，边缘缓存RTMP会很麻烦。如果是HTTP，缓存其实也很麻烦，但是HTTP服务器的缓存已经做了很久，所以只需要使用就好。这是为何点播都走HTTP的原因


### 4、对比互联网上用的流媒体分发方式

- HLS：apple的HLS，支持点播和直播
- HTTP：即HTTP stream，各家自己定义的http流，应用于国内点播视频网站
- RTMP：直播应用，对实时性有一定要求，以PC为主
