# PublishAotCompressed.macOS

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![NuGet](https://img.shields.io/badge/NuGet-1.0.0--preview-blue)](https://www.nuget.org/packages/PublishAotCompressed.macOS)

在 macOS 上自动使用 UPX 压缩 Native AOT 二进制文件的 MSBuild 目标。专为 macOS 交叉编译到 Windows 和 Linux 设计，无缝集成。

[English](README.md) | 简体中文

## 🚀 特性

- ✅ **自动 UPX 压缩** - Native AOT 编译后自动压缩
- ✅ **交叉编译支持** - 从 macOS 压缩 Windows 和 Linux 二进制文件
- ✅ **macOS 专用** - 为 macOS 开发工作流优化
- ✅ **60%+ 体积减小** - 典型压缩率
- ✅ **可选 LZMA** - 更好的压缩效果
- ✅ **智能检测** - 自动跳过 macOS 目标的压缩

## 📋 目录

- [快速开始](#快速开始)
  - [Windows 交叉编译](#windows-交叉编译)
  - [Linux 交叉编译](#linux-交叉编译)
  - [macOS 原生编译](#macos-原生编译)
- [配置](#配置)
- [支持的目标平台](#支持的目标平台)
- [工作原理](#工作原理)
- [压缩效果](#压缩效果)
- [故障排查](#故障排查)
- [相关项目](#相关项目)

## 快速开始

### 前置要求

- **macOS** (Apple Silicon 或 Intel)
- **.NET 9.0 SDK** 或更高版本
- **UPX** 通过 Homebrew 安装：
  ```bash
  brew install upx
  ```

### Windows 交叉编译

本包与 [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) 完美配合，用于交叉编译到 Windows。

1. **安装交叉编译工具**：
   ```bash
   # 安装 LLVM 以获取 lld-link
   brew install lld
   
   # 添加到 PATH（或添加到 ~/.zshrc 永久生效）
   export PATH="$(brew --prefix lld)/bin:$PATH"
   
   # 安装 xwin 以获取 Windows SDK
   cargo install --locked xwin
   
   # 下载 Windows SDK (~1.5GB)
   mkdir -p $HOME/.local/share/xwin-sdk
   xwin --accept-license \
     --cache-dir $HOME/.local/share/xwin-sdk \
     --arch x86_64,aarch64 \
     splat --preserve-ms-arch-notation
   ```

2. **添加包到项目**：
   ```xml
   <ItemGroup>
     <PackageReference Include="PublishAotCross.macOS" Version="1.0.3-preview" />
     <PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-preview" />
   </ItemGroup>
   ```

3. **发布并压缩**：
   ```bash
   # 确保 lld-link 在 PATH 中
   export PATH="$(brew --prefix lld)/bin:$PATH"
   
   # 为 Windows 构建（自动使用 UPX 压缩）
   dotnet publish -r win-x64 -c Release
   dotnet publish -r win-arm64 -c Release
   dotnet publish -r win-x86 -c Release
   ```

📖 **详细 Windows 设置指南**：参见 [PublishAotCross.macOS QUICKSTART.md](https://github.com/interface95/PublishAotCross.macOS/blob/main/QUICKSTART.md)

### Linux 交叉编译

1. **安装 Zig**（通过 Homebrew）：
   ```bash
   brew install zig
   ```

2. **添加包到项目**（同上）：
   ```xml
   <ItemGroup>
     <PackageReference Include="PublishAotCross.macOS" Version="1.0.3-preview" />
     <PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-preview" />
   </ItemGroup>
   ```

3. **发布并压缩**：
   ```bash
   # 基于 glibc 的系统（Ubuntu、Debian 等）
   dotnet publish -r linux-x64 -c Release /p:StripSymbols=false
   dotnet publish -r linux-arm64 -c Release /p:StripSymbols=false
   
   # 基于 musl 的系统（Alpine Linux）
   dotnet publish -r linux-musl-x64 -c Release /p:StripSymbols=false
   dotnet publish -r linux-musl-arm64 -c Release /p:StripSymbols=false
   ```

📖 **详细 Linux 设置指南**：参见 [PublishAotCross.macOS QUICKSTART-LINUX.md](https://github.com/interface95/PublishAotCross.macOS/blob/main/QUICKSTART-LINUX.md)

### macOS 原生编译

**好消息**：macOS 目标是内置的！只需正常发布：

```bash
dotnet publish -r osx-arm64 -c Release
dotnet publish -r osx-x64 -c Release
```

> ✅ **注意**：macOS 目标会**自动跳过 UPX 压缩**。包会显示一条消息并生成未压缩的可正常运行的二进制文件。这是设计使然，因为 UPX 压缩的 macOS 二进制文件由于 macOS 安全限制（系统完整性保护、代码签名、Gatekeeper）无法运行。

## 配置

### 压缩设置

您可以使用完整属性名或短别名：

```xml
<PropertyGroup>
  <!-- 短属性名（推荐） -->
  <Upx>true</Upx>
  
  <!-- 或使用完整属性名 -->
  <PublishAotCompressed>true</PublishAotCompressed>
  
  <!-- 使用最佳压缩（默认：true，压缩等级 9） -->
  <CompressBest>true</CompressBest>
  
  <!-- 使用 LZMA 获得最大压缩（启动较慢） -->
  <PublishLzmaCompressed>true</PublishLzmaCompressed>
</PropertyGroup>
```

### 命令行用法

```bash
# 启用压缩（短格式）✅ 推荐
dotnet publish -r win-x64 -c Release /p:Upx=true

# 禁用压缩（短格式）✅ 推荐  
dotnet publish -r win-x64 -c Release /p:Upx=false

# 或使用完整属性名
dotnet publish -r win-x64 -c Release /p:PublishAotCompressed=false
```

### 项目配置示例

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <PublishAot>true</PublishAot>
    
    <!-- 推荐的体积优化选项 -->
    <UseSystemResourceKeys>true</UseSystemResourceKeys>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
  
  <ItemGroup>
    <!-- macOS 交叉编译 + UPX 压缩 -->
    <PackageReference Include="PublishAotCross.macOS" Version="1.0.3-preview" />
    <PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-preview" />
  </ItemGroup>
</Project>
```

## 支持的目标平台

### Windows（通过 lld-link + xwin）

| 目标 | UPX 支持 | 交叉编译工具 |
|------|---------|------------|
| `win-x64` | ✅ 压缩 | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `win-arm64` | ✅ 压缩 | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `win-x86` | ✅ 压缩 | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |

### Linux（通过 Zig）

| 目标 | UPX 支持 | 交叉编译工具 |
|------|---------|------------|
| `linux-x64` | ✅ 压缩 | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `linux-arm64` | ✅ 压缩 | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `linux-musl-x64` | ✅ 压缩 | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `linux-musl-arm64` | ✅ 压缩 | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |

### macOS（原生）

| 目标 | UPX 支持 | 备注 |
|------|---------|------|
| `osx-arm64` | ⏭️ 跳过 | 由于 macOS 安全限制自动跳过 |
| `osx-x64` | ⏭️ 跳过 | 由于 macOS 安全限制自动跳过 |

## 工作原理

本包钩入 Native AOT 构建过程，自动压缩编译后的二进制文件：

**构建流程：**

```
.NET AOT 编译器（macOS）→ 原生二进制文件 → UPX 压缩 → 压缩后的可执行文件
                          (.exe/.elf)      (macOS 工具)   (目标操作系统)
```

**技术细节：**

1. **MSBuild 目标**：在链接后立即运行 `AfterTargets="LinkNative"` 来压缩二进制文件
2. **平台检测**：从 RuntimeIdentifier 识别目标操作系统（win-*/linux-*/osx-*）
3. **智能压缩**：
   - 对于 Windows/Linux 目标：使用 `--best` 标志运行 UPX
   - 对于 macOS 目标：跳过压缩并显示消息
4. **主机工具**：始终使用包中包含的 macOS ARM64 UPX 二进制文件

**UPX 压缩：**
- **算法**：LZBA（默认）或 LZMA（可选）
- **等级**：默认 `--best`（等级 9）
- **解压缩**：程序启动时自动在内存中解压，通常察觉不到

## 压缩效果

对于启用了体积优化的 Hello World 程序：

| 目标 | 未压缩 | 使用 `--best` UPX | 压缩率 | 节省空间 |
|------|--------|------------------|--------|---------|
| **Windows x64** | 1.05 MB | 483 KB | 44.9% | 55.1% |
| **Windows ARM64** | 1.01 MB | 455 KB | 45.0% | 55.0% |
| **Linux x64** | 1.30 MB | 520 KB | 40.0% | 60.0% |
| **Linux ARM64** | 1.25 MB | 500 KB | 40.0% | 60.0% |
| **macOS ARM64** | 1.20 MB | N/A（跳过） | - | - |

> 💡 **提示**：添加 `<PublishLzmaCompressed>true</PublishLzmaCompressed>` 可获得更好的压缩效果（通常小 5-10%，但启动时间增加约 50-100ms）。

## 部署到目标平台

### Linux 部署

.NET Native AOT 二进制文件在目标系统上需要 **ICU 库**：

```bash
# Ubuntu/Debian
sudo apt-get install -y libicu-dev

# CentOS/RHEL/Fedora
sudo yum install -y icu

# Alpine Linux
apk add --no-cache icu-libs
```

**Docker 示例：**

```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y libicu-dev
COPY YourApp /app/
CMD ["/app/YourApp"]
```

**禁用 ICU（可选）：**

```xml
<PropertyGroup>
  <InvariantGlobalization>true</InvariantGlobalization>
</PropertyGroup>
```

### Windows 部署

UPX 压缩的 Windows 可执行文件：
- ✅ 在任何 Windows 10/11 系统上运行
- ✅ 无需额外运行时（.NET Native AOT）
- ⚠️ 某些杀毒软件可能误报（需要时添加到白名单）

**如果 Windows 显示"压缩文件"对话框：**
- 这是 WinRAR/7-Zip 的文件关联问题
- **解决方案**：右键 → 属性 → 解除锁定（如果显示）
- `.exe` 是直接可执行文件，不是压缩包

## 故障排查

### `upx: command not found`

通过 Homebrew 安装 UPX：

```bash
brew install upx
```

### 交叉编译失败

确保已安装交叉编译工具：
- Windows：参见 [PublishAotCross.macOS Windows 设置](https://github.com/interface95/PublishAotCross.macOS#for-windows-cross-compilation)
- Linux：参见 [PublishAotCross.macOS Linux 设置](https://github.com/interface95/PublishAotCross.macOS#for-linux-cross-compilation)

### UPX 压缩失败

检查包是否正确安装：

```bash
# 验证 UPX 可用
upx --version

# 检查包安装
dotnet list package | grep PublishAotCompressed.macOS
```

### "PublishAotCompressed.macOS can only be used on macOS"

本包只能在 macOS 上使用。其他平台请使用：
- 原始跨平台版本：[PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed)

## 相关项目

### 完整的 .NET Native AOT 交叉编译生态系统

| 项目 | 用途 | 平台 |
|------|------|------|
| **[PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS)** | 从 macOS 交叉编译 | macOS → Windows/Linux |
| **PublishAotCompressed.macOS**（本项目） | macOS 上的 UPX 压缩 | 压缩 Windows/Linux 二进制文件 |
| [PublishAotCross](https://github.com/agocke/PublishAotCross) | 从 Windows 交叉编译 | Windows → Linux |
| [PublishAotCrossXWin](https://github.com/Windows10CE/PublishAotCrossXWin) | 从 Linux 交叉编译 | Linux → Windows |
| [PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed) | 原始 UPX 包 | 多平台 |

### 交叉编译 + 压缩矩阵

| 构建主机 | 目标 | 交叉编译 | UPX 压缩 |
|---------|------|---------|---------|
| **macOS** | Windows | ✅ PublishAotCross.macOS | ✅ 本包 |
| **macOS** | Linux | ✅ PublishAotCross.macOS | ✅ 本包 |
| **macOS** | macOS | 原生 | ⏭️ 跳过（安全限制） |
| Windows | Linux | ✅ PublishAotCross | ✅ PublishAotCompressed |
| Linux | Windows | ✅ PublishAotCrossXWin | ✅ PublishAotCompressed |

> 💡 **macOS 用户两全其美** - 在一台机器上交叉编译到 Windows 和 Linux，并自动进行 UPX 压缩！

## 要求

### 构建主机
- **macOS**（Apple Silicon 或 Intel）
- **.NET 9.0 SDK** 或更高版本
- **Homebrew**（用于安装工具）

### Windows 交叉编译
- **LLVM**（`lld-link` 链接器）- 通过 `brew install lld`
- **Rust/Cargo**（用于安装 xwin）- 通过 `brew install rust`
- **xwin** - 通过 `cargo install xwin`
- **约 1.5GB 磁盘空间**用于 Windows SDK

### Linux 交叉编译
- **Zig**（约 200MB，包含所有内容）- 通过 `brew install zig`

## 示例项目

参见本仓库中的 `test/` 目录获取完整示例。

## 许可证

MIT License - 详见 [LICENSE.TXT](LICENSE.TXT)

## 致谢

- 基于 Michal Strehovsky 的 [PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed)
- UPX 由 [UPX 团队](https://upx.github.io/)开发
- 专为与 [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) 配合使用而设计

## 贡献

欢迎贡献！请随时提交 Pull Request。

## 支持

- 🐛 **问题**：[GitHub Issues](https://github.com/interface95/PublishAotCompressed.macOS/issues)
- 💬 **讨论**：[GitHub Discussions](https://github.com/interface95/PublishAotCompressed.macOS/discussions)

---

**❤️ 为 macOS 上的 .NET Native AOT 社区制作**

