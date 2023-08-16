# JavaCV1.5.5以后版本拉流hevc(H265)编码的实视频流崩溃，切换到1.5.5以下或者1.4x版本或者更低版本后却正常拉流的问题解决办法

 问题描述：
JavaCV1.5.5，1.5.6，1.5.7以及更高版本中，拉流含有h265(hevc)编码的视频直接grabber.start()崩溃，切换到1.5.5以下或者1.4x版本或者更低版本后却正常拉流的问题。

故障原因
是由于缺少GPLv2的依赖问题。
这个问题博主在之前的GPLv2排雷文章中已经提到过
 JavaCV的gpl v2许可协议排雷，写在TikTok违反GPLv2许可使用OBS源码的当下，JavaCV1.5.5及更高版本，把全部GPLv2授权的开源库分离到了单独的GPL依赖，不需要用到这些库的开发者可以直接忽略，但是用到的，比如h265

FFmpeg中涉及到的部分GPLv2的库：

```
avisynth
frei0r
libcdio
libdavs2
librubberband
libvidstab
libx264
libx265
libxavs
libxavs2
libxvid
```

如何解决
增加以下gpl依赖即可（以1.5.6版本的javacv为例）：

```
<!-- Optional GPL builds with (almost) everything enabled -->
    <dependency>
        <groupId>org.bytedeco</groupId>
        <artifactId>ffmpeg-platform-gpl</artifactId>
        <version>4.4-1.5.6</version>
    </dependency>
```

end


-----------------------------------
©著作权归作者所有：来自51CTO博客作者eguid的原创作品，请联系作者获取转载授权，否则将追究法律责任
JavaCV1.5.5以后版本拉流hevc(H265)编码的实视频流崩溃，切换到1.5.5以下或者1.4x版本或者更低版本后却正常拉流的问题解决办法
https://blog.51cto.com/eguid/5729120