# mpv-armbian

针对 ARM Linux 平台（Armbian / 单板计算机）优化 mpv 播放器的 Debian 包构建流程。所有依赖库从源码静态编译并嵌入 libmpv，集成 **V4L2 Request API** 硬件视频解码，在 ARM 开发板上实现低功耗高性能媒体播放。

## 构建产出的 Debian 包

| 包名 | 内容 | 文件示例 |
|------|------|----------|
| `mpv` | mpv 播放器主程序 + 文档 | `/usr/bin/mpv`, `/usr/share/*` |
| `libmpv2` | libmpv 运行时动态库 | `/usr/lib/<arch>/libmpv.so.*` |
| `libmpv-dev` | libmpv 开发文件 | 头文件、pkgconfig、`libmpv.so` 符号链接 |

> 所有第三方依赖均以**静态库**形式链接进 libmpv，运行时无需单独安装。

## 目标平台

| 平台 | 架构 | 硬件解码 |
|------|------|----------|
| 树莓派 3/4/5 (Raspberry Pi) | aarch64 | V4L2 Request API (H.264/MPEG2/VP9/HEVC) |
| 香橙派 (Orange Pi) | aarch64 | V4L2 Request API + Cedar |
| Rockchip 开发板 | aarch64 | V4L2 Request API |
| Amlogic S905 系列 | aarch64 | V4L2 Request API |
| x86_64 PC | amd64 | VAAPI + VDPAU + NVDEC |

## 核心特性

### V4L2 Request API 硬件加速

通过两个自定义补丁为 ARM 平台添加基于 Linux 内核 V4L2 Request API 的硬件视频解码：

- **`patches/ffmpeg-v4l2request.patch`** — 为 FFmpeg 新增 `h264_v4l2request`、`hevc_v4l2request`、`mpeg2_v4l2request`、`vp9_v4l2request` 等硬件解码器
- **`patches/mpv-v4l2request.patch`** — 为 mpv 新增 `v4l2-request` 硬件解码驱动，配合 `vo_dmabuf_wayland` 实现零拷贝渲染

### mpv 构建选项

| 启用 | 禁用 |
|------|------|
| Wayland + X11 + XPresent、PulseAudio | Vulkan |
| libmpv 共享库（`-Dlibmpv=true`）、CLI 播放器（`-Dcplayer=true`） | 单元测试（`-Dtests=false`）、编译时间戳（`-Dbuild-date=false`） |

## 构建依赖链

```
harfbuzz ──┐
            ├──→ libass ──────────────────────┐
fribidi  ──┘                                  │
                                              │
dav1d ───────────────→ ffmpeg ────────────────┤
                          ↑                   │
nv-codec-headers ─────────┘ [仅 amd64]        ├──→ mpv
                                              │
hwdata ──→ libdisplay-info                    │
                   │                          │
                   └──→ libplacebo ───────────┘
```

## 环境要求

### 构建主机依赖

```bash
sudo apt-get install -y --no-install-recommends \
  debhelper cmake build-essential ninja-build nasm \
  python3-setuptools wget git ca-certificates \
  xorg-dev libxpresent-dev libwayland-dev libxkbcommon-dev \
  wayland-protocols libegl-dev libv4l-dev \
  libpulse-dev libuchardet-dev \
  libva-dev libvdpau-dev libudev-dev \
  libdrm-dev libgbm-dev libssl-dev \
  liblua5.2-dev
```

> Meson 需要 1.9+ 版本，CI 中从 [GitHub release](https://github.com/mesonbuild/meson/archive/1.9.0.tar.gz) 手动安装，而非使用 apt 仓库版本。

### 目标系统

- **发行版**: Ubuntu 22.04+、Debian 12+
- **内核**: Linux 5.15+（V4L2 Request API 需 5.15+，Allwinner 平台建议 6.1+）
- **GPU 驱动**: Panfrost / Lima / V3D 等开源 Mesa 驱动

## 构建

### 第一步：编译依赖（静态库安装到 /usr/local）

```bash
make -f debian/rules depends
```

该目标按依赖链依次编译 dav1d、nv-codec-headers（仅 amd64）、harfbuzz、fribidi、libass、FFmpeg（自动应用 V4L2 补丁）、libplacebo、libdisplay-info，最后拉取 mpv 源码并应用 `patches/mpv-v4l2request.patch`。

### 第二步：打包 .deb

```bash
dpkg-buildpackage -b -nc
```

| 参数 | 说明 |
|------|------|
| `-b` | 仅构建二进制包，跳过源码包 |
| `-nc` | 跳过 `make clean`，避免删除已编译的依赖 |

### 一键构建

```bash
make -f debian/rules depends && dpkg-buildpackage -b -nc
```

## 安装

生成的 `.deb` 文件位于当前目录的上级：

```bash
# 校验
mpv --version
pkg-config --modversion mpv
```

## 使用 V4L2 硬件解码

```bash
# 查看可用的硬件解码器
mpv --hwdec=help

# 使用 V4L2 Request API 硬件解码播放
mpv --hwdec=v4l2-request video.mp4

# 配合 Wayland DMA-BUF 零拷贝渲染
mpv --hwdec=v4l2-request --vo=gpu-next video.mp4
```

## CI/CD

GitHub Actions（`.github/workflows/build.yaml`）在 push 到 `armbian` 分支时触发，构建矩阵为 **Ubuntu 22.04 / 24.04、Debian 12 / 13** × **x86_64 / aarch64**（共 8 种组合）。

流程：安装依赖（含源码安装 Meson 1.9.0）→ `make -f debian/rules depends` → `dpkg-buildpackage -b -nc` → 上传 `.deb` 至 Artifacts；失败时附带上传构建日志。

## License

本仓库构建脚本以 [GNU General Public License v3.0](LICENSE) 发布。mpv（GPLv2+）、FFmpeg（LGPL/GPL）等上游项目使用各自的许可证。
