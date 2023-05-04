#iOS 图片加载和处理

## 前言

本文基于[WWDC2018-Image and Graphics Best Practices](https://developer.apple.com/videos/play/wwdc2018/219/)，对图片加载和处理的思考和总结

## 正文

图片的显示分为三步：加载、解码、渲染。

通常，我们的操作只有加载，解码和渲染是又UIKit完成。

![图片加载解码渲染](https://upload-images.jianshu.io/upload_images/1049769-8863f93fef3541b1.png)

## 什么是解码

以UIImageView为例。当其显示在屏幕上时，需要UIImage作为数据源。

UIImage持有的数据是未解码的压缩数据，能节省较多的内存和加快存储。（解码就是解压缩，我个人理解）

当UIImage被赋值给UIImageView的时候（imageView.image = image），图像数据会被解码，变成RGB的颜色数据。

解码是一个计算量较大的任务，并且需要CPU来完成；且解码出来的图片体积与图片的宽度哟关系，而与图片原来的体积无关。

其体积大小可简单描述为：宽 * 高 * 每个像素点的大小 = width * height * 4bytes。

![解码](https://upload-images.jianshu.io/upload_images/1049769-01f9bc2691d7b400.png)

## 图像解码操作会造成什么问题？

以我们常见的UITableView和UICollectionView为例，加入我们在使用一个多图片显示的功能：

![多图示例](https://upload-images.jianshu.io/upload_images/1049769-776e375772ebb4a3.png)

在上下滑动显示图片的过程中，我们会在CellForRow的方法加载UIImage图片、赋值给UIImageView，相当于在主线程同事进行IO操作、解码操作等，会造成内存迅速增长和CPU的负担。如果系统无法满足做够的内存，则会先结束其他后台进程，最终还是无法满足时，将会结束当前进程。

![](https://upload-images.jianshu.io/upload_images/1049769-66339448b2f54c47.png)

## 那么如何针对这种情况进行优化？

### 优化1：降采样

在滑动显示过程中，图片显示的宽度远比真是图片要小，我们可以采用加载缩略图的方式减少图片的占用内存。

如下图所示：

![](https://upload-images.jianshu.io/upload_images/1049769-c89b5af960cfdb1b.png)

我们加载JPEG图片，然后进行相关设置，解码后根据设置成成CGImage缩略图，最后包装成UIImage，最终传递给UIImageView渲染。

**思考：这里的解码步骤为何不是上文提的imageView.image= image时机？**

```
func downsample(imageAt imageURL: URL, to porintSize: CGSize, Scale: CGFloat) -> UIImage {
	let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
	let imageSource = CGImageSourceCreateWithURL(imageURL as CFURL, ImageSourceOptions)!
	let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale
	let downsampleOptoins = [kCGImageSourceCreateThumbnailFromImageAlways: true,
	kkCGImageSourceShouldCacheImmediately: true,
	kCGImageSourceCreateThumbnailWithTransform: true,
	kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels] as CFDictionary
	let downsampledImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, 	downsampleOptions)!
	return UIImage(cgImage: downsampledImage)]
}

```

我的理解：正常的UIImage加载是从app本地读取，或者从网络下载图片，此时不涉及图片内容相关的操作，并不需要解码；当图片被赋值给UIImageView的时候，CALayer读取图片内容进行渲染，所需要对图片进行解码；而上文的缩略图生成过程中，已经对图片进行解码操作，此时的UIImage只是一个CGImage的封装，所以当UIImage赋值给UIImageView时，CALayer可以直接使用CGImage所持有的图像数据。

### 优化2：异步处理

![](https://upload-images.jianshu.io/upload_images/1049769-0f126b13d55f8595.png)

从用户的体验来分析，滑动的操作往往是间断性触发，在滑动的瞬间有较大的工作量，而且由于都在主线程进行操作无法进行任务分配，CPU2处于闲置状态，由此引申出两种优化手段：

* prefetching（预处理）
* background decoding/downsample(子线程解码和降采样)。

综合起来，可以在prefetching的时候把降采样放到子线程进行处理，因为降采样过程包括解码操作。

![](https://upload-images.jianshu.io/upload_images/1049769-adc5d8f6870f9d47.png)

Prefetching回调中，把降采样的操作放到同步队列serialQueue中，处理完毕之后抛给主线程进行update操作。

需要特别注意，此处不能是并发队列，否则会造成线程爆炸，原因见总结部分。

![](https://upload-images.jianshu.io/upload_images/1049769-5f0ca58951a7a96e.png)

### 优化3：使用image asset catalogs

Apple推荐的图片资源管理工具，压缩效率更高，在iOS 12的机器上有10~20%的空间节约，并且每个版本Apple都会持续对其进行优化。
内容较多，详细可点[Session](https://developer.apple.com/videos/play/wwdc2018/227)。

## 总结

应用上述的优化策略，已经能对图片加载有比较好的优化。
WWDC后续还有对CustomDrawing和CALayer的BackingStore的介绍，因为与图片关系不大，不在此赘述。

下面再介绍我对WWDC学习的看法。

文章转载于

>作者：落影loyinglin
>
>链接：https://www.jianshu.com/p/7d8a82115060
>
>来源：简书
>
>著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。