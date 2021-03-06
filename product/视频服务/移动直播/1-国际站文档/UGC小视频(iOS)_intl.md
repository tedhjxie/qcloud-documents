
## Overview 
As end users have more and more customization demands, simple text interaction and image upload can no longer satisfy their needs for presenting and sharing their messages. This is when UGC (User Generated Content) short videos are born. RTMP SDK 2.0.0 supports recording and publishing UGC short videos (the editing feature is still in development). This document describes how to use this feature.

## Interfacing Process
The process of recording and publishing UGC short videos is divided into the following three steps:

![](//mc.qcloudimg.com/static/img/283c8d7fe0a5a316097ae687a2bf6c5a/image.png)

* Step 1: Use the TXUGCRecord API to record a short video. When the record is finished, a short video file (MP4) is returned to the user;

* Step 2: Your App applies for an upload signature from your business server. The upload signature is a "license" which allows the App to upload MP4 file to Tencent Cloud Video Distribution Platform. To ensure safety, these signatures are issued by the server and can only be used once.

* Step 3: Use the TXUGCPublish API to publish video. Once successful, the SDK will return the playback URL to you while Tencent Cloud Video Distribution Platform will ensure implementation of certain video playback capabilities such as closest location scheduling, instant video launching, dynamic acceleration and overseas access, which can only be realized through huge investments.

## Note

- App must not include the SecretID and SecretKey of the upload signature in its client code. Exposing these critical information will cause potential safety issues such as allowing attackers to use your traffic and storage service for free once they have cracked your App and obtained the information.

- The proper practice is to use the SecretID and SecretKey on your server to generate a one-time upload signature, then provide the signature to the App. Usually, servers are difficult to crack so this will ensure safety.

- Be sure to transmit the correct SecretID and Signature fields when you publish short videos or the publish operation will fail;

- When calling the video recording APIs startRecord and stopRecord, make sure they're called in pairs;

- Video recording length is determined by your code, to keep optimal performance and a good user experience, it is recommended that you keep the recording length within 60 seconds.


## API Description 
RTMP SDK provides relevant APIs used for recording and publishing short videos. Detailed definitions are listed below:

|  API File |  Feature |
| ------| --------|
| `TXUGCRecord.h` | Record short videos |
| `TXUGCPublish.h`` | Upload/publish short videos |
| `TXUGCRecordListener.h` | Record callback and publish callback for short videos |
| `TXUGCRecordTypeDef.h` | Basic parameter definitions |

## Interfacing Guide

![](//mc.qcloudimg.com/static/img/6b21b033259c1b5124648b73e88fb243/image.png)


### 1. Video Preview
TXUGCRecord (located in TXUGCRecord.h) is used to record short videos. Our first task is to realize the video preview feature. The startCamera function is used to launch preview. Launching preview also needs to open the camera and microphone, thus you may receive permission prompt.

```ObjectiveC
UIView *    preViewContainer;                    //Prepare a view used for previewing camera image
TXUGCSimpleConfig *config = [[TXUGCSimpleConfig alloc] init];
//config.videoQuality = VIDEO_QUALITY_LOW;       // For 360p, a video of 10 seconds is around 0.75 MB
config.videoQuality   = VIDEO_QUALITY_MEDIUM;    // For 540p, a video of 10 seconds is around 1.5 MB (encoding parameters are the same with Wechat iOS short videos)
//config.videoQuality = VIDEO_QUALITY_HIGH;      // For 720p, a video of 10 seconds is around 3 MB
config.watermark      = image;                   // Watermark image (use a PNG image with transparent background)
config.watermarkPos   = pos;                     // Watermark image position
config.frontCamera    = YES;                     //Whether to use front camera. Use switchCamera to switch camera
[TXUGCRecord shareInstance].delegate = self;     //TXVideoPublishListener API is realized by self
[[TXUGCRecord shareInstance] startCamera:param preview:preViewContainer];
```

### 2. Special Effect
You can use the special effect switches in TXUGCRecord to add certain special effects to your video or control the camera (either before or during a recording process).

```ObjectiveC
//////////////////////////////////////////////////////////////////////////
//                      Supported special effects are shown below (1.9.1 version or above)
//////////////////////////////////////////////////////////////////////////
//
// Switch between front and rear camera. The parameter isFront indicates whether to use front camera. Default is front camera
[[TXUGCRecord shareInstance] switchCamera YES];

// Configure beatify and whitening effect levels
// beautyDepth     : Available value range for beautify level: 0-9. 0 means to disable beautify, while a higher value means a stronger effect.
// whiteningDepth  : Available value range for whitening level: 0-9. 0 means to disable whitening, while a higher value means a stronger effect.
[[TXUGCRecord shareInstance] setBeautyDepth: 7 WhiteningDepth: 1];

// Configure color filter: romantic, fresh, artistic, tender, vintage...
// image     : Specify the color lookup table used by the filter. Note: Be sure to use PNG format!
// The filter lookup table image used in the Demo is located in RTMPiOSDemo/RTMPiOSDemo/resource/FilterResource.bundle
// setSpecialRatio: Used to configure the effect level of the filter (0-1). A higher value means a stronger effect. Default is 0.5
[[TXUGCRecord shareInstance] setFilter: filterImage];
[[TXUGCRecord shareInstance] setSpecialRatio: 0.5];

// Whether to enable flashlight
[[TXUGCRecord shareInstance] toggleTorch: YES];

//////////////////////////////////////////////////////////////////////////
//                       The following special effects are only supported by the VIP edition
// (These effects used intellectual properties of the Youtu team and are supported only by the VIP SDK edition. We cannot provide them for free)
//////////////////////////////////////////////////////////////////////////

// Configure eye enlargement level (0-9); ----- It is recommended to let the VJs choose their desired effect level, as different people have different tastes
[[TXUGCRecord shareInstance] setEyeScaleLevel: 0];

// Configure face slimming level (0-9); ----- It is recommended to let the VJs choose their desired effect level, as different people have different tastes
[[TXUGCRecord shareInstance] setFaceScaleLevel: 0];

// Configure motion sticker  tmplName - material name   tmplDir - path of the material package
[[TXUGCRecord shareInstance] selectMotionTmpl: tmplName inDir: tmplDir];

// Configure green screen effect. Green screen material is an MP4 file with an aspect ratio of 9:16 (for example, 368*640, 540*960, 720*1280)
[[TXUGCRecord shareInstance] setGreenScreenFile: file]; 
```


### 3. Record File
Call the startRecord function in TXUGCRecord to start recording process, and call the stopRecord function to stop recording process. Be sure to call these two functions in pairs.
```ObjectiveC
[[TXUGCRecord shareInstance] startRecord];
[[TXUGCRecord shareInstance] stopRecord];
``` 

The feedback of the recording process and result is provided by the TXVideoRecordListener API (defined in TXUGCRecordListener.h):

- onRecordProgress is used to provide feedback of the progress of recording process, while the parameter millisecond indicates record length (in milliseconds):
```ObjectiveC
@optional
-(void) onRecordProgress:(NSInteger)milliSecond;
``` 

- onRecordComplete provides feedback of the result of recording process. The fields retCode and descMsg indicate error code and error message, videoPath is the path of the short video file, while coverImage is the first frame image of the short video (automatically captured, this image can be used when you publish the video).
```ObjectiveC   
@optional
-(void) onRecordComplete:(TXRecordResult*)result;
```     

### 4. Preview File
Use the [Playback SDK](https://cloud.tencent.com/document/product/454/7880) to preview the generated MP4 file. When calling startPlay, you need to specify playback type as [PLAY_TYPE_LOCAL_VIDEO](https://cloud.tencent.com/document/product/454/7880#step-3.3A-.E5.90.AF.E5.8A.A8.E6.92.AD.E6.94.BE6).

### 5. Acquire Signature
To publish the generated MP4 file onto Tencent Cloud Video Distribution CDN, you need **SecretID** and **Signature**. Similar to user name and password, they are used to ensure the safety of your cloud storage service and prevent attackers from stealing your traffic and storage.

- **SecretID (Key ID)**
You can obtain or create a SecretID in the management section of [Cloud API Key](https://console.cloud.tencent.com/capi), as shown in the figure below (marked in red box):
![](//mc.qcloudimg.com/static/img/23f95aaa97adf3eeae3bf90470fe5122/image.png)

- **Signature (Upload Signature)**
The upload signature is a one-time string calculated using a standard signature algorithm, based on the SecretID and SecretKey you obtained from Tencent Cloud.

 To ensure safety, you need to put the signature calculation program in your backend server instead of putting the calculation function into the App, because it's easy to crack an App and acquire the SecretKey used for the signature, while cracking a server is an impossible task for most attackers.

 For more information about the method for calculating signature, please see [How to Generate Signature?](https://cloud.tencent.com/document/product/266/7835?!preview&lang=zh#.E8.8E.B7.E5.8F.96.E7.AD.BE.E5.90.8D.E8.AE.A1.E7.AE.97.E6.89.80.E9.9C.80.E4.BF.A1.E6.81.AF). You can leave the fields FileName, FileSha and uid empty when generating the publish signature.

### 6. Publish File
TXUGCPublish (located in TXUGCPublish.h) is used to publish MP4 files onto the Tencent Cloud Video Distribution Platform in order to satisfy video playback demands such as closest location scheduling, instant video launching, dynamic acceleration and overseas access.

```ObjectiveC
TXPublishParam * param = [[TXPublishParam alloc] init];

param.secretId  = @"AKIDeqtlGihED4oqjRP2324seJn1313MLnaa";   // You need to enter your SecretId
param.signature = _signature;                                // You need to enter the upload signature calculated in step 4

// You can obtain the path of the generated video file from the onRecordComplete callback of ITXVideoRecordListener
param.videoPath = _videoPath;  
// You can obtain the first-frame preview image of the generated video file from the onRecordComplete callback of ITXVideoRecordListener
param.coverPath = _coverImage; 

TXUGCPublish *_ugcPublish = [[TXUGCPublish alloc] init];
_ugcPublish.delegate = self;                                 // Configure TXVideoPublishListener callback
[_ugcPublish publishVideo:param];
``` 

The feedback of the publishing process and result is provided by the TXVideoPublishListener API (defined in the TXUGCRecordListener.h header file):

- onPublishProgress provides feedback of the f
- ile publishing progress, the parameter uploadBytes indicates bytes currently uploaded, while the parameter totalBytes indicates total bytes to be uploaded.
```ObjectiveC 
@optional
-(void) onPublishProgress:(NSInteger)uploadBytes totalBytes: (NSInteger)totalBytes;
```

- onPublishComplete provides feedback of publishing result. The fields errCode and descMsg of TXPublishResult indicate error code and error message. videoURL is the playback address of the short video. coverURL is the cloud storage address of the video cover image. videoId is the cloud storage ID of the video file, you can use this ID to call VOD [Server APIs](https://cloud.tencent.com/document/product/266/1965).
``` C 
@optional
-(void) onPublishComplete:(TXPublishResult*)result;
```
