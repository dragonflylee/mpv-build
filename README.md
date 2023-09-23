# mingw64-packages

用于 MSYS2/MinGW-w64 环境的 mpv 播放器全套构建包。从源码编译 mpv (libmpv) 及其全部 13 个依赖库，支持 Windows 平台。

## 架构支持

| 架构 | MSYSTEM | 说明 |
|------|---------|------|
| x86_64 | `mingw64` | 64 位 Windows (GCC) |
| i686 | `mingw32` | 32 位 Windows (GCC) |
| aarch64 | `clangarm64` | 64 位 ARM Windows (LLVM/Clang) |

## 构建产物

mpv 构建为 **libmpv 动态库**，所有依赖库构建为**静态库**并链接到 libmpv 中。

关键特性：
- **D3D11 DXGI 渲染后端** — 通过自定义补丁 `0001-d3d11-render.patch` 添加，支持通过 `MPV_RENDER_API_TYPE_DXGI` 将 mpv 视频输出直接渲染到 D3D11 纹理
- **硬件加速** — FFmpeg 启用 D3D11VA、DXVA2、NVDEC 等硬件解码
- **Windows 8+ 兼容** — `_WIN32_WINNT` 设为 `0x0602`，DDPI API 动态加载以兼容旧系统

## 环境要求

### 构建工具（MSYS2 包）

```bash
pacman -S --needed \
  ${MINGW_PACKAGE_PREFIX}-cc \
  ${MINGW_PACKAGE_PREFIX}-meson \
  ${MINGW_PACKAGE_PREFIX}-ninja \
  ${MINGW_PACKAGE_PREFIX}-pkgconf \
  ${MINGW_PACKAGE_PREFIX}-nasm
```

## 快速开始

### 构建全部包

构建顺序会按依赖关系自动处理：`uchardet → mbedtls → freetype → harfbuzz → libass → dav1d → ffmpeg → spirv-headers → spirv-tools → glslang → spirv-cross → shaderc → libplacebo → mpv`

```bash
# 进入对应 MSYSTEM 终端（如 MINGW64），然后：
makepkg-mingw -sciCf --noconfirm
```

## 自定义补丁

| 补丁 | 作用 |
|------|------|
| `mpv/0001-d3d11-render.patch` | 添加 D3D11 DXGI 渲染后端，新增 `render_dxgi.h` 和 `libmpv_d3d11.c`，降低 Windows 版本要求 |
| `shaderc/0001-fix-glslang-hlsl-linking-order.patch` | 修复 shaderc pkgconfig 文件，添加 SPIRV-Tools/glslang 链接依赖 |
| `spirv-cross/0001-static-linking-hacks.patch` | 将 spirv-cross-c-shared 从动态库改为静态库 |
| `uchardet/0001-fix-pkgconfig-files.patch` | 修复 uchardet pkgconfig 中的绝对路径 |

## CI/CD

通过 GitHub Actions (`.github/workflows/mingw.yaml`) 自动构建：

- 在 `mingw64`、`mingw32`、`clangarm64` 三个平台并行构建
- 按依赖顺序依次编译所有 14 个包
- 将生成的 `.pkg.tar.zst` 文件发布为 prerelease

## License

- 本仓库脚本以 [Apache License 2.0](LICENSE) 发布
- mpv 及各依赖库使用各自的许可证（GPL、LGPL 等）
