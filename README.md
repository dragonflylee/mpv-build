# mpv-portlibs

为 Nintendo Switch (Horizon OS) 构建 [mpv](https://mpv.io/) 及其移植依赖库的 PKGBUILD 集合，基于 [devkitPro](https://devkitpro.org/) 工具链。产物为带 `nvtegra` 硬件解码、`deko3d` 渲染和 `audren` 音频的静态 `libmpv`。

## 特性

- **硬件加速**：FFmpeg 集成 [`envideo`](https://github.com/averne/envideo)（Tegra X1 NVDEC/NVJPG），提供 `h264_envideo`、`hevc_envideo` 等硬件解码器及 `deinterlace_envideo` / `denoise_envideo` / `sharpness_envideo` 滤镜。
- **deko3d 渲染**：mpv 提供基于 `libuam` 的 deko3d 视频输出。
- **Horizon 音频**：`ao_hos` 驱动，使用 `audren`。
- **网络串流**：curl + mbedtls + FFmpeg `https/tls/http` 协议。

## 包列表

| 目录 | 包名 | 说明 |
| --- | --- | --- |
| `dav1d/` | `switch-dav1d` | AV1 解码器 |
| `envideo/` | `envideo` | Nvidia 多媒体引擎底层封装（NVDEC 硬解） |
| `mbedtls/` | `switch-mbedtls` | TLS/SSL 库 |
| `curl/` | `switch-curl` | HTTP(S) 客户端 |
| `ffmpeg/` | `switch-ffmpeg` | 集成 nvtegra 的 FFmpeg |
| `libuam/` | `libuam` | deko3d 着色器编译器 |
| `mpv/` | `switch-libmpv` / `switch-libmpv-deko3d` | mpv 库及 deko3d 变体 |
| `nspmini/` | `switch-nspmini` | NSP 安装库 |
| `hacbrewpack/` | `hacBrewPack` | NCA/NSP 打包工具 |

## 构建

```sh
docker build -t mpv-portlibs .
docker run --rm -it -v "$PWD":/work mpv-portlibs bash

# 容器内按依赖顺序构建
cd /work
for pkg in dav1d mbedtls curl libuam envideo ffmpeg mpv; do
    ( cd "$pkg" && makepkg -si --noconfirm )
done
```

`switch-libmpv`（GL 输出）与 `switch-libmpv-deko3d`（deko3d 输出）互相冲突，按需选择。

## 许可

构建脚本遵循 [Apache License 2.0](LICENSE)。FFmpeg 与 mpv 以 GPL 构建，分发二进制时须遵守 GPL 条款。
