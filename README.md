# Video.js for web.blink

[Video.js](https://github.com/videojs/video.js) 修改版，用于 web.blink( Miniblink 内核 ) 扩展库。  

之前写过在 Miniblink 中调用 HTML5Media 播放视频的方法，  
参考相关文章：[《调用各种浏览器控件播放视频》](https://mp.weixin.qq.com/s/p3_Fz25MlHKgdURrHk9FlA)  
参考相关源码：https://github.com/aardio/html5media-blink  

这次我们改用更流行的 Video.js 来播放视频，这里还是有几个坑要填：  
首先只支持 Video.js 7.7.4 这一个版本，更高更低版本都不行。对于之前文章提到的 Miniblink 检测 video 节点的坑，  
在 Video.js 中可以通过在源码中直接改 defaultTechOrder_ 解决，当然这个在参数里也可以改。  

另外 Miniblink 无法正常触发全屏事件，document.onfullscreenchange 无效，webkitfullscreenchange 也是没用的。  
要解决这个问题只能改 Video.js 的源码直接调用 aardio 的全屏函数。  

为了实现超级懒人包，请先升级 web.blink 扩展库，  
以下是调用范例（ aardio 代码 ）：

```javascript
import web.blink.form;
import web.npPlugin.flash;
import wsock.tcp.asynHttpServer;
import win.ui;
/*DSG{{*/
var winform = win.form(text="web.blink 支持 HTML5 视频（基于 Flash）";right=1008;bottom=616)
winform.add()
/*}}*/

var httpServer = wsock.tcp.asynHttpServer(); 
httpServer.run( {
	["/index.html"] = /**
<html>
<head>
	<style type="text/css">
		html,body { height:100%;width:100%;margin:0;overflow:hidden; }
	</style>
	
	<!--第一步：引入下面3个文件加载播放器 --> 
	<link href="https://cdn.jsdelivr.net/gh/aardio/video-js-blink/video-js.min.css" rel="stylesheet">
	<script src="https://cdn.jsdelivr.net/gh/aardio/video-js-blink/video.min.js"></script> 
	<script src="https://cdn.jsdelivr.net/npm/videojs-flash@2.2.1/dist/videojs-flash.min.js"></script> 
</head>
<body>

	<!-- 第二步：插入视频文件 -->
	<video id="my-player" class="video-js" controls preload="auto" style="width:100%;height:100%"  >
		<source src="http://download.aardio.com/demo/video.aardio" type="video/mp4"></source>
	</video>

	<script>
			
		//第三步：初始化播放器,这一步不能省略
		var obj = videojs("my-player", { 
			controlBar: {
				fullscreenToggle: true
			}
		}); 
		
	</script>
</body>
</html>
**/;
}); 

var wb = web.blink.form( winform );
wb.go( httpServer.getUrl("index.html") );

winform.show(); 
win.loopMessage();
```
