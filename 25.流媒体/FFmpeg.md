## FFmpeg

[FFmpeg官网](https://ffmpeg.org/download.html)

网站中的FFMPEG分为3个版本：Static，Shared，Dev，区别在于：    
&emsp;&emsp;Static里面只有3个应用程序：ffmpeg.exe，ffplay.exe，ffprobe.exe，每个exe的体积都很大，相关的Dll已经被编译到exe里面去了；        
&emsp;&emsp;Shared里面除了3个应用程序：ffmpeg.exe，ffplay.exe，ffprobe.exe之外，还有一些Dll，比如说avcodec-54.dll之类的，Shared里面的exe体积很小，他们在运行的时候，到相应的Dll中调用功能；       
&emsp;&emsp;Dev版本是用于开发的，里面包含了库文件xxx.lib以及头文件xxx.h，这个版本不包含exe文件；

### 1、ffmpeg
ffmpeg是用于转码的应用程序
```ruby
// 将input.avi转码成output.ts，并设置视频的码率为640kbps
ffmpeg -i input.avi -b:v 640k output.ts
```
### 2、ffplay
ffplay是用于播放的应用程序
```ruby
// 播放test.avi
ffplay test.avi
```
### 3、ffprobe
ffprobe是用于查看文件格式的应用程序

