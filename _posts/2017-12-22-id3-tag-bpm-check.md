---
title: "ID3 태그에 BPM 메타 확인"
date: 2017-12-22 11:11:56
tags:
- ffmpeg
- id3tag
- bpm
---

# Goal

Check ID3 BPM tags using ffmpeg

ID3v2 버전부터 태그에 Text information frames 안에 TBPM 필드가 추가되었다.

- TBPM

  The 'BPM' frame contains the number of beats per minute in the mainpart of the audio. The BPM is an integer and represented as a numerical string. 

  ​

> 확인한 ffmpeg version 및 라이브러리 (구 버전의 ffmpeg으로는 TBPM 정보를 확인할 수 없었다.)
>
> $ ffmpeg -version

```shell
ffmpeg version 3.3.2 Copyright (c) 2000-2017 the FFmpeg developers
  built with Apple LLVM version 8.1.0 (clang-802.0.42)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/3.3.2 --enable-shared --enable-pthreads --enable-gpl --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-libass --enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-opencl --disable-lzma --enable-nonfree --enable-vda
```



#### sample

```shell
Input #0, mp3, from 'Battle_Symphony.mp3':
  Metadata:
    title           : Battle Symphony
    artist          : Linkin Park
    album           : One More Light
    genre           : Pop
    track           : 4
    TBPM            : 149.05
    album_artist    : Linkin Park

```



#### ID3v2메타 정보가 없을경우

```
$ aubio tempo Battle_Symphony.mp3
[mp3 @ 0x7faf07812200] Could not update timestamps for skipped samples.
148.31 bpm
```

