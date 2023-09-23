# mingw64-packages

在 MSYS2/MinGW-w64 环境中从源码构建 mpv (libmpv) 及其全部依赖库的 PKGBUILD 集合。

## 支持的架构

| 架构 | MSYSTEM | 工具链 |
|------|---------|--------|
| x86_64 | `mingw64` | GCC |
| i686 | `mingw32` | GCC |
| aarch64 | `clangarm64` | LLVM/Clang |

## 产物说明

- **mpv**：以 `--default-library=shared` 构建为动态 `libmpv`（`-Dlibmpv=true -Dcplayer=false`），禁用 Vulkan/libarchive，启用 D3D11。
- **依赖库**：构建为静态库，通过 `--prefer-static` 链接进 libmpv。
- **Windows 7+ 兼容**：通过 `win7-compat.patch` 将 `_WIN32_WINNT` 降至 `0x0602`，并动态加载 `GetDpiForMonitor`、`AdjustWindowRectExForDpi`、`GetSystemMetricsForDpi` 等高版本 API。

## 依赖与构建顺序

共 16 个包，按依赖顺序构建：

```
mbedtls → curl → libwebp → freetype → harfbuzz → fribidi → libass
→ dav1d → ffnvcodec-headers → ffmpeg → spirv-headers → spirv-tools
→ glslang → spirv-cross → shaderc → libplacebo → mpv
```

## 环境要求

```bash
pacman -S --needed \
  patch unzip make git diffutils \
  ${MINGW_PACKAGE_PREFIX}-cc \
  ${MINGW_PACKAGE_PREFIX}-meson \
  ${MINGW_PACKAGE_PREFIX}-cmake \
  ${MINGW_PACKAGE_PREFIX}-nasm
```

## 构建

在对应的 MSYSTEM 终端（如 MINGW64）中，按依赖顺序逐包执行：

```bash
for pkg in mbedtls curl libwebp freetype harfbuzz fribidi libass dav1d \
  ffnvcodec-headers ffmpeg spirv-headers spirv-tools glslang spirv-cross \
  shaderc libplacebo mpv; do
  pushd "$pkg" && makepkg-mingw -sciCf --noconfirm && popd
done
```

## 自定义补丁

| 包 | 补丁 | 作用 |
|----|------|------|
| mpv | `win7-compat.patch` | Windows 7 兼容：降低 `_WIN32_WINNT` 并动态加载高版本 DPI/窗口 API |
| ffmpeg | `avio.patch`（远程） | 新增 libssh2/SFTP 协议，并通过 `FFMPEG_STATIC_DLLEXPORT` 重新导出 AVIO API 供 libmpv 使用 |
| curl | `static-libs.patch` | 仅构建/使用静态库 |
| libwebp | `0001-mingw-cmake-output.patch` | 修正 MinGW CMake 输出 |
| shaderc | `0001-fix-glslang-hlsl-linking-order.patch` | 修复 pkgconfig 中的 glslang/SPIRV-Tools 链接顺序 |
| spirv-cross | `0001-static-linking-hacks.patch` | 将 spirv-cross-c-shared 改为静态库 |

## CI

GitHub Actions（`.github/workflows/mingw.yaml`）在 `mingw64`、`mingw32`、`clangarm64` 三个平台上并行构建：

- 依次编译全部包，上传所有 `.pkg.tar.zst` 作为构建产物
- 将 `curl`、`libwebp`、`mpv` 的包发布到 `mingw-package` prerelease

## License

- 本仓库构建脚本以 [Apache License 2.0](LICENSE) 发布。
- mpv 及各依赖库遵循各自许可证（GPL、LGPL 等）。
