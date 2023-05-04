# iOS下音视频的编码

## 音视频方案的取舍

* 自研团队
	* 资金雄厚
	* 修炼技术
* 第三方商业SDK
	*  项目团队小，需要快速介入

## 案例说明

### 直播APP

 * 主播
 	* 音视频的采集（AVFoundation）
 	* 美颜/滤镜 
 		* coreImage 使用的非常少，难用
 		* GPUImage
 			*  开源
 			*  集成采集业务-> AVFoundation
 			*  有三个版本
 				* 第一、二个基于OpenGLES （使用的语言是GLSL）
 				* 第三个基于Metal 正在不断的更新中 （Metal language）
 			* 120多种滤镜
 				* 灵活
 				* 自定义滤镜
 			* OpenCV
 				* 识别： 人脸识别，文字识别
 			* OpenGL
 				* 渲染、绘制
 				* 离屏渲染
 	* 3 编码 （压缩）
	 	* 音频
	 		* AAC
	 	* 视频
	 		* H264
	 * 4 推流 （LFLiveKit框架）
* 观众
	* 拉流
	* 解码
		* ijkplayer
	* 渲染
 				

