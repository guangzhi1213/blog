---
title: "Ffmpeg"
date: 2021-04-25T12:30:18+08:00
lastmod: 2021-04-25T12:30:18+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## ffmpeg

- [雷霄骅音视频技术笔记](https://blog.csdn.net/leixiaohua1020)
- [罗索实验室](http://www.rosoo.net/)
- [ffmpeg 中国论坛](http://bbs.chinaffmpeg.com/)

截取视频上半, 镜像

`ffmpeg.exe -i INPUT -vf "split [main][tmp]; [tmp] crop=iw:ih/2:0:0, vflip [flip]; [main][flip] overlay=0:H/2" OUTPUT`

mp4转ts分片

命令：

`ffmpeg -i xxx.mp4 -f segment -segment_time 60 -segment_format mpegts -segment_list /home/higherlevel/video-folder/video_name.m3u8 -c copy -bsf:v h264_mp4toannexb -map 0 /home/higherlevel/video-folder/course-%04d.ts`

命令行规则: 

1. 相同的Filter线性链之间用逗号分隔
2. 不同的Filter线性链之间用分号分隔

命令行参数:

`-f` 输出文件的格式

工作流程:

1. 解封装(Demuxing)
2. 解码(Decoding)
3. 编码(Encoding)
4. 封装(Muxing)

## ffplay

播放器

提供了音视频显示和播放相关的图像信息, 音频的波形信息等.

## ffprobe

多媒体分析器
