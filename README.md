# mpv-dylib

macOS 平台下构建 mpv 动态库（libmpv.dylib）的编译脚本。将 mpv 及其所有依赖从源码编译为动态链接库，适用于嵌入 mpv 播放能力的 macOS 应用开发。

## 架构支持

| 架构 | 部署目标 | Meson 配置 |
|------|----------|------------|
| x86_64 (Intel) | macOS 10.15 | `x86_64.meson` |
| arm64 (Apple Silicon) | macOS 11.0 | `arm64.meson` |

默认编译架构为 `x86_64`。

## 环境要求

- **操作系统**: macOS 10.15+
- **构建工具**:
  - Xcode Command Line Tools（提供 clang）
  - [CMake](https://cmake.org/)（>= 3.15）
  - [Ninja](https://ninja-build.org/)
  - [Meson](https://mesonbuild.com/)（>= 0.60）
  - [pkg-config](https://www.freedesktop.org/wiki/Software/pkg-config/)

### 安装构建工具

```bash
# 使用 Homebrew 安装
brew install cmake ninja meson pkg-config
```

确保 Xcode Command Line Tools 已安装：

```bash
xcode-select --install
```

## 快速开始

### 编译 x86_64 架构（默认）

```bash
make
```

### 编译 arm64 架构

```bash
make OSX_ARCH=arm64
```

### 编译指定架构

```bash
# 仅下载依赖源码
make download

# 编译 x86_64
make build OSX_ARCH=x86_64

# 编译 arm64
make build OSX_ARCH=arm64
```

### 清理

```bash
make clean
```

## 输出

编译完成后，产物位于 `dylib/<架构>/` 目录下：

```
dylib/<arch>/
├── include/          # 头文件
│   └── mpv/
│       └── client.h
└── lib/
    ├── libmpv.dylib          # mpv 动态库
    ├── libavcodec.dylib      # FFmpeg 编解码库
    ├── libavformat.dylib     # FFmpeg 封装格式库
    ├── libavutil.dylib       # FFmpeg 工具库
    ├── libswscale.dylib      # FFmpeg 图像缩放
    ├── libswresample.dylib   # FFmpeg 音频重采样
    └── ...                   # 其他依赖库
```

## 自定义补丁

`mpv.patch` 对 mpv 源码做了以下修改，使其在禁用 Swift 的情况下（仅构建 libmpv 时）也能正常编译：

- 将 Cocoa 媒体键、剪贴板等依赖 Swift 的功能用 `HAVE_COCOA && HAVE_SWIFT` 条件保护
- 移除了仅与播放器界面相关的 `clipboard-mac.m` 等文件的编译依赖

## License

mpv 及其依赖库使用各自的许可证。本仓库仅提供构建脚本，按原样提供。
