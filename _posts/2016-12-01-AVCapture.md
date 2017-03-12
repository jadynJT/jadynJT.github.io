---
layout:     post
title:      使用AVCapture
subtitle:   AVCapture
date:       2016-12-01
author:     JT
header-img: img/post-bg-hengda1.jpg
catalog: true
tags:
    - iOS

---

# 前言
>可用于音频、二维码、拍照、录制视频 （均可自定义界面）

## 常见的输出信号
- `AVCaptureAudioDataOutput` 音频输出
- `AVCaptureFileOutput `文本输出
- 	`AVCaptureMetadataOutput `二维码 条形码…
-  `AVCaptureStillImageOutput` 拍照
-  `AVCaptureMovieFileOutput` 录制视频（不能实现暂停录制和定义视频文件类型）
-  `AVCaptureVideoDataOutput + AVCaptureAudioDataOutput` 录制视频的灵活性更强（能实现暂停录制和定义视频文件类型）

## 举例
>**使用AVCaptureMovieFileOutput输出流实现视频录制**

**初始化会话层**

```
-(void)sessionConfiguration{

    //初始化一个会话
    session = [[AVCaptureSession alloc] init];
    [session setSessionPreset:AVCaptureSessionPresetMedium];

    //创建视频设备
    AVCaptureDevice *videoDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];

    //根据设备创建输入信号
    deviceInput = [AVCaptureDeviceInput deviceInputWithDevice:videoDevice error:nil];

    //添加 输出设备 movieFile
    self.deviceMovieFileOutput = [[AVCaptureMovieFileOutput alloc] init];

    [session beginConfiguration];
    //session添加设备输入信号
    if ([session canAddInput:deviceInput]) {
        [session addInput:deviceInput];
    }
    //session添加设备输出信号
    if ([session canAddOutput:self.deviceMovieFileOutput]) {
        [session addOutput:self.deviceMovieFileOutput];
    }
    [session commitConfiguration];
}

```

**创建预览图层**

```
-(void)embedLayerWithView:(UIView *)view{
    if (session == nil) {
        return;
    }
    videoPreviewLayer = [AVCaptureVideoPreviewLayer layerWithSession:session];
    //设置图层的大小
    videoPreviewLayer.frame = view.bounds;
    videoPreviewLayer.videoGravity = AVLayerVideoGravityResizeAspectFill;
    [view.layer addSublayer:videoPreviewLayer];
    [session startRunning];
}

```

**录制视频**

```
-(void)takePhoto:(NSURL *)fileURL{
    [self.deviceMovieFileOutput startRecordingToOutputFileURL:fileURL recordingDelegate:self];
}

```

**结束录制**

```
-(UIImageView *)finishRecord:(UIView *)view isAnewRecording:(BOOL)anewRecording{
    gifImageView = [[UIImageView alloc] initWithFrame:view.bounds];
    [view addSubview:gifImageView];
    isAnewRecording = anewRecording; //存储是否重新录制
    //停止录制(停止录制后做代理方法)
    [self.deviceMovieFileOutput stopRecording];
    return gifImageView;
}

```

**拍摄视频保存路径**

```
+(NSString *)getVideoSaveFilePath{
    NSString*documentPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filePath = [documentPath stringByAppendingPathComponent:@"video.mp4"];
    return filePath;
}

```

**会话层启动和关闭**

```

-(void)startCamera{
    [session startRunning];
}

-(void)stopCamera{
    [session stopRunning];
}

```
**代理方法**

```
- (void)captureOutput:(AVCaptureFileOutput *)captureOutput didFinishRecordingToOutputFileAtURL:(NSURL *)outputFileURL fromConnections:(NSArray *)connections error:(NSError *)error{

    NSLog(@"完成录制");
    NSLog(@"outputFileURL = %@",outputFileURL);

    //**重新录制**//
    if (isAnewRecording) {
        //**删除视频文件**//
        NSFileManager *manager = [NSFileManager defaultManager];
        [manager removeItemAtPath:outputFileURL.absoluteString error:nil];
    }
    //**不取消录制**//
    else{
        //**获取视频时长**//
        AVURLAsset *avUrl = [AVURLAsset URLAssetWithURL:outputFileURL options:nil];
        CMTime time = [avUrl duration];
        int seconds = ceil(time.value/time.timescale);

        NSLog(@"seconds = %d",seconds);

        if ([self.delegate respondsToSelector:@selector(videoDuration:)]) {
            [self.delegate videoDuration:seconds];
        }
        if ([self.delegate respondsToSelector:@selector(playerVideo:)]) {
            [self.delegate playerVideo:outputFileURL.absoluteString];
        }
    }
}

```



