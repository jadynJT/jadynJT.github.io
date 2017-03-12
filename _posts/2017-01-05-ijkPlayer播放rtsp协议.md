---
layout:     post
title:      ijkPlayer播放rtsp协议
subtitle:   ijkPlayer、rtsp
date:       2017-01-05
author:     JT
header-img: img/post-bg-basa1.jpg
catalog: true
tags:
    - iOS

---

# 前言
>由于FFmpeg的config文件默认没有开启对rtsp协议的支持，所以导致rtsp的地址一直无法播放

## 解决方法
- 来到ijkplayer/config目录下，找到module-lite.sh文件，该文件是编译FFmpeg的配置文件  
  
- 打开该文件，找到  
  `export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-protocol=rtp" `   
  修改为以下，就可以打开rtsp协议了  
  `export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtp"`  
  
- 打开rtsp音视频分离器  
`export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=rtsp"`  

- 执行以下命令，连接配置文件，开始编译  
  
  `cd config `  
  `rm module.sh `  
  `ln -s module-lite.sh module.sh`   
  `cd iOS`  
  `./compile-ffmpeg.sh clean`   
  `./compile-ffmpeg.sh all`
  
  
## 无法播放 
  
 >编译完成后发现rtsp协议已经支持，但可能仍播放不了，并可看到错误信息  
 >`No codec could be found with id 8`
 
 重新打开**module-lite.sh**文件，添加以下两行  
 `export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-decoder=mjpeg"`  
 `export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=mjpeg"`
 
 重新编译，打包Framework，发现可以正常播放，但是实际播放效果不理想，卡顿严重！
 
## 优化ijkPlayer
>添加相应的配置代码，使其达到最佳的播放效果


```
IJKFFOptions *options = [IJKFFOptions optionsByDefault];

    [options setPlayerOptionIntValue:30  forKey:@"max-fps"];
    [options setPlayerOptionIntValue:1  forKey:@"framedrop"];
    [options setPlayerOptionIntValue:0  forKey:@"start-on-prepared"];
    [options setPlayerOptionIntValue:0  forKey:@"http-detect-range-support"];
    [options setPlayerOptionIntValue:48  forKey:@"skip_loop_filter"];
    [options setPlayerOptionIntValue:0  forKey:@"packet-buffering"];
    [options setPlayerOptionIntValue:2000000 forKey:@"analyzeduration"];
    [options setPlayerOptionIntValue:25  forKey:@"min-frames"];
    [options setPlayerOptionIntValue:1  forKey:@"start-on-prepared"];

    [options setCodecOptionIntValue:8 forKey:@"skip_frame"];

    [options setFormatOptionValue:@"nobuffer" forKey:@"fflags"];
    [options setFormatOptionValue:@"8192" forKey:@"probsize"];
    [options setFormatOptionIntValue:0 forKey:@"auto_convert"];
    [options setFormatOptionIntValue:1 forKey:@"reconnect"];

    [options setPlayerOptionIntValue:1  forKey:@"videotoolbox"];

    // 帧速率(fps) （可以改，确认非标准桢率会导致音画不同步，所以只能设定为15或者29.97）
    [options setPlayerOptionIntValue:29.97 forKey:@"r"];
    // -vol——设置音量大小，256为标准音量。（要设置成两倍音量时则输入512，依此类推
    [options setPlayerOptionIntValue:512 forKey:@"vol"];

    _player = [[IJKFFMoviePlayerController alloc] initWithContentURL:self.url withOptions:options];

```
 
 


