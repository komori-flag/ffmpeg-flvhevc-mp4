# FFmpeg — FLV HEVC (CodecID 12) → MP4 Minimal Build

这个仓库是官方 FFmpeg 的精简构建配置，专用于将**国内厂商使用非标准 CodecID 12 封装 HEVC 的 FLV 文件**转封装为 MP4。

## 用法

```bash
# 流复制（最快，无需重新编码）
ffmpeg -i input.flv -c copy output.mp4
```

## 构建矩阵

通过 GitHub Actions 自动编译 6 个平台版本：

| 产物 | 平台 | 编译方式 |
|---|---|---|
| `ffmpeg-flvhevc-mp4-win-x86_64.exe` | Windows x86_64 | Ubuntu + MinGW 交叉编译 |
| `ffmpeg-flvhevc-mp4-linux-x86_64` | Linux glibc x86_64 | Ubuntu 原生编译 |
| `ffmpeg-flvhevc-mp4-linux-aarch64` | Linux glibc ARM64 | Ubuntu + ARM64 交叉编译 |
| `ffmpeg-flvhevc-mp4-linux-x86_64-musl` | Linux musl 全静态 | Alpine 容器编译 |
| `ffmpeg-flvhevc-mp4-macos-arm64` | macOS Apple Silicon | macOS 原生编译 |
| `ffmpeg-flvhevc-mp4-macos-x86_64` | macOS Intel | macOS 原生编译 |

所有版本均约 **1-2 MB**，只保留了 FLV demuxer + MP4 muxer + HEVC/AAC parser + bitstream filter，**不含解码器/编码器/滤镜**。

## 精简配置

```bash
--disable-everything           # 关掉一切
--enable-ffmpeg                # 只开 ffmpeg CLI
--enable-demuxer=flv           # FLV 读取（含 CodecID 12 → HEVC）
--enable-muxer=mp4             # MP4 写入
--enable-parser=hevc           # HEVC 帧边界检测
--enable-parser=aac            # AAC 帧边界检测
--enable-bsf=hevc_mp4toannexb  # HEVC 转 MP4 格式
--enable-bsf=aac_adtstoasc     # AAC 转 MP4 格式
--enable-small                 # 尺寸优化
--disable-decoders             # 纯转封装，不需要解码器
--disable-encoders             # 不需要编码器
--disable-swscale              # 不需要图像缩放
--disable-swresample           # 不需要重采样
--disable-devices              # 不需要设备
--disable-filters              # 不需要滤镜
```

## 原理

国内部分厂商在 FLV 容器中使用 **Video Tag CodecID = 12** 表示 HEVC（H.265），但此值非 FLV 官方规范。FFmpeg 官方（master 分支）已在 `libavformat/flvdec.c` 中内置对此非标准扩展的支持，映射 `FLV_CODECID_X_HEVC = 12` → `AV_CODEC_ID_HEVC`。

此构建仅需 **FLV demuxer + MP4 muxer + parser + bitstream filter**，无需解码器参与，通过流复制（`-c copy`）完成转封装。

## License

本仓库中的构建脚本和配置文件采用 **GNU General Public License v3.0**。
构建产出的 `ffmpeg.exe` / `ffprobe.exe` 基于 FFmpeg 源码编译，遵循 **GPL v2+**（因 fftools CLI 代码为 GPL）。
使用或分发二进制时请遵守对应许可证条款。

---

# FFmpeg README

FFmpeg is a collection of libraries and tools to process multimedia content
such as audio, video, subtitles and related metadata.

## Libraries

* `libavcodec` provides implementation of a wider range of codecs.
* `libavformat` implements streaming protocols, container formats and basic I/O access.
* `libavutil` includes hashers, decompressors and miscellaneous utility functions.
* `libavfilter` provides means to alter decoded audio and video through a directed graph of connected filters.
* `libavdevice` provides an abstraction to access capture and playback devices.
* `libswresample` implements audio mixing and resampling routines.
* `libswscale` implements color conversion and scaling routines.

## Tools

* [ffmpeg](https://ffmpeg.org/ffmpeg.html) is a command line toolbox to
  manipulate, convert and stream multimedia content.
* [ffplay](https://ffmpeg.org/ffplay.html) is a minimalistic multimedia player.
* [ffprobe](https://ffmpeg.org/ffprobe.html) is a simple analysis tool to inspect
  multimedia content.
* Additional small tools such as `aviocat`, `ismindex` and `qt-faststart`.

## Documentation

The offline documentation is available in the **doc/** directory.

The online documentation is available in the main [website](https://ffmpeg.org)
and in the [wiki](https://trac.ffmpeg.org).

### Examples

Coding examples are available in the **doc/examples** directory.

## License

FFmpeg codebase is mainly LGPL-licensed with optional components licensed under
GPL. Please refer to the LICENSE file for detailed information.

## Contributing

Patches should be submitted to the ffmpeg-devel mailing list using
`git format-patch` or `git send-email`. Github pull requests should be
avoided because they are not part of our review process and will be ignored.
