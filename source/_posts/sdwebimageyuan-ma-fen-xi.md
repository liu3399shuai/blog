---
layout: post
title: "SDWebImage源码分析"
date: 2016-02-21 22:49:32 +0800
comments: true
tags:
- [sdwebimage]
categories:
- [图片下载]
- [OC]
---

带有缓存的异步图片下载器，这个库提供了一个UIImageView的类别，这个类别支持远程图片资源加载并显示，源码链接[SDWebImage](https://github.com/rs/SDWebImage)

<!-- more -->

### what is it ?

带有缓存的异步图片下载器，这个库提供了一个UIImageView的类别，这个类别支持远程图片资源加载并显示，源码链接[SDWebImage](https://github.com/rs/SDWebImage)
 
### 框架结构

如图

![](/images/sdwebimage.jpg)

![](/images/sdwebimage_ifelse.png)

从它的文件夹分类看，有这么几层

* 资源下载及管理资源下载
* 资源缓存
* UI层使用资源，以类别形式体现，引入头文件，直接使用

#### 资源下载

```
SDWebImageDownloaderOperation.h
```
这个类封装一个完整的下载任务 : 继承于NSOperation，在里面创建一个用于下载的NSURLConnection，绑定下载需要的request，打开当前线程runloop，实现NSURLConnection的代理方法

```
SDWebImageDownloader.h

```
这个类负责下载管理器downloaderManager : 对外部下载的入口封装 ，利用自己的operationQueue 将每个用于下载创建的operation进行管理

可以把它看做下图的功能

![](/images/downloadmanager.jpg)


### 资源缓存

```
SDImageCache.h
```
这个类负责缓存文件管理 : 增加新的文件、删除存储的文件、查找存储的文件、计算、清除 存储的容量等、设定存储容量，文件数量等

例如

```
[[SDImageCache sharedImageCache] storeImage:myImage forKey:myCacheKey];

```

```
SDImageCache *imageCache = [[SDImageCache alloc] initWithNamespace:@"myNamespace"];
[imageCache queryDiskCacheForKey:myCacheKey done:^(UIImage *image) {
    // image is not nil if image was found
}];

```

### 管理

```
SDWebImageManager.h
```
这个类是 webimage 任务的总入口，集成 下载部分 + 缓存部分

```
SDWebImageManager *manager = [SDWebImageManager sharedManager];
[manager downloadImageWithURL:imageURL
                      options:0
                     progress:^(NSInteger receivedSize, NSInteger expectedSize) {
                         // progression tracking code
                     }
                     completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
                         if (image) {
                             // do something with image
                         }
                     }];
```

### 用户使用层

都是以category形式对类进行方法扩展，直接调用即可

```
#import <SDWebImage/UIImageView+WebCache.h>

...

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString *MyIdentifier = @"MyIdentifier";

    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:MyIdentifier];
    if (cell == nil) {
        cell = [[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault
                                       reuseIdentifier:MyIdentifier] autorelease];
    }

    // Here we use the new provided sd_setImageWithURL: method to load the web image
    [cell.imageView sd_setImageWithURL:[NSURL URLWithString:@"http://www.domain.com/path/to/image.jpg"]
                      placeholderImage:[UIImage imageNamed:@"placeholder.png"]];

    cell.textLabel.text = @"My Text";
    return cell;
}
```

或者这样

```
// Here we use the new provided sd_setImageWithURL: method to load the web image
[cell.imageView sd_setImageWithURL:[NSURL URLWithString:@"http://www.domain.com/path/to/image.jpg"]
                      placeholderImage:[UIImage imageNamed:@"placeholder.png"]
                             completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
                                ... completion code here ...
                             }];
```

### 图示流

![](/images/webimage_flow.png)










