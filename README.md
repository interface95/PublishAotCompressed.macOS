# PublishAotCompressed.macOS

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![NuGet](https://img.shields.io/badge/NuGet-1.0.0--preview-blue)](https://www.nuget.org/packages/PublishAotCompressed.macOS)

MSBuild targets to automatically compress Native AOT binaries with UPX on macOS. Designed to work seamlessly with cross-compilation from macOS to Windows and Linux.

English | [简体中文](README.zh-CN.md)

## 🚀 Features

- ✅ **Automatic UPX compression** after Native AOT compilation
- ✅ **Cross-compilation support** - compress binaries for Windows and Linux from macOS
- ✅ **macOS-only** - streamlined for macOS development workflow
- ✅ **60%+ size reduction** - typical compression rates
- ✅ **Optional LZMA** for even better compression
- ✅ **Smart detection** - automatically skips compression for macOS targets

## 📋 Table of Contents

- [Quick Start](#quick-start)
  - [For Windows Cross-compilation](#for-windows-cross-compilation)
  - [For Linux Cross-compilation](#for-linux-cross-compilation)
  - [For macOS (native)](#for-macos-native)
- [Configuration](#configuration)
- [Supported Targets](#supported-targets)
- [How It Works](#how-it-works)
- [Compression Results](#compression-results)
- [Troubleshooting](#troubleshooting)
- [Related Projects](#related-projects)

## Quick Start

### Prerequisites

- **macOS** (Apple Silicon or Intel)
- **.NET 9.0 SDK** or later
- **UPX** installed via Homebrew:
  ```bash
  brew install upx
  ```

### For Windows Cross-compilation

This package works perfectly with [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) for cross-compiling to Windows.

1. **Install cross-compilation tools**:
   ```bash
   # Install LLVM for lld-link
   brew install lld
   
   # Add to PATH (or add to ~/.zshrc for permanent)
   export PATH="$(brew --prefix lld)/bin:$PATH"
   
   # Install xwin for Windows SDK
   cargo install --locked xwin
   
   # Download Windows SDK (~1.5GB)
   mkdir -p $HOME/.local/share/xwin-sdk
   xwin --accept-license \
     --cache-dir $HOME/.local/share/xwin-sdk \
     --arch x86_64,aarch64 \
     splat --preserve-ms-arch-notation
   ```

2. **Add packages to your project**:
   ```xml
   <ItemGroup>
     <PackageReference Include="PublishAotCross.macOS" Version="1.0.3-preview" />
     <PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-preview" />
   </ItemGroup>
   ```

3. **Publish with compression**:
   ```bash
   # Ensure lld-link is in PATH
   export PATH="$(brew --prefix lld)/bin:$PATH"
   
   # Build for Windows (automatically compressed with UPX)
   dotnet publish -r win-x64 -c Release
   dotnet publish -r win-arm64 -c Release
   dotnet publish -r win-x86 -c Release
   ```

📖 **Detailed Windows setup guide**: See [PublishAotCross.macOS QUICKSTART.md](https://github.com/interface95/PublishAotCross.macOS/blob/main/QUICKSTART.md)

### For Linux Cross-compilation

1. **Install Zig** (via Homebrew):
   ```bash
   brew install zig
   ```

2. **Add packages to your project** (same as above):
   ```xml
   <ItemGroup>
     <PackageReference Include="PublishAotCross.macOS" Version="1.0.3-preview" />
     <PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-preview" />
   </ItemGroup>
   ```

3. **Publish with compression**:
   ```bash
   # glibc-based (Ubuntu, Debian, etc.)
   dotnet publish -r linux-x64 -c Release /p:StripSymbols=false
   dotnet publish -r linux-arm64 -c Release /p:StripSymbols=false
   
   # musl-based (Alpine Linux)
   dotnet publish -r linux-musl-x64 -c Release /p:StripSymbols=false
   dotnet publish -r linux-musl-arm64 -c Release /p:StripSymbols=false
   ```

📖 **Detailed Linux setup guide**: See [PublishAotCross.macOS QUICKSTART-LINUX.md](https://github.com/interface95/PublishAotCross.macOS/blob/main/QUICKSTART-LINUX.md)

### For macOS (native)

**Good news**: macOS targets are built-in! Just publish normally:

```bash
dotnet publish -r osx-arm64 -c Release
dotnet publish -r osx-x64 -c Release
```

> ✅ **Note**: macOS targets automatically **skip UPX compression**. The package will display a message and produce uncompressed binaries that run normally. This is by design since UPX-compressed macOS binaries cannot run due to macOS security restrictions (System Integrity Protection, code signing, Gatekeeper).

## Configuration

### Compression Settings

You can use either the full property name or the short alias:

```xml
<PropertyGroup>
  <!-- Short property name (recommended) -->
  <Upx>true</Upx>
  
  <!-- Or use the full property name -->
  <PublishAotCompressed>true</PublishAotCompressed>
  
  <!-- Use best compression (default: true, compression level 9) -->
  <CompressBest>true</CompressBest>
  
  <!-- Use LZMA for maximum compression (slower startup) -->
  <PublishLzmaCompressed>true</PublishLzmaCompressed>
</PropertyGroup>
```

### Command Line Usage

```bash
# Enable compression (short form) ✅ Recommended
dotnet publish -r win-x64 -c Release /p:Upx=true

# Disable compression (short form) ✅ Recommended  
dotnet publish -r win-x64 -c Release /p:Upx=false

# Or use the full property name
dotnet publish -r win-x64 -c Release /p:PublishAotCompressed=false
```

### Example Project Configuration

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <PublishAot>true</PublishAot>
    
    <!-- Recommended size optimizations -->
    <UseSystemResourceKeys>true</UseSystemResourceKeys>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
  
  <ItemGroup>
    <!-- Cross-compilation + UPX compression for macOS -->
    <PackageReference Include="PublishAotCross.macOS" Version="1.0.3-preview" />
    <PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-preview" />
  </ItemGroup>
</Project>
```

## Supported Targets

### Windows (via lld-link + xwin)

| Target | UPX Support | Cross-compilation Tool |
|--------|-------------|----------------------|
| `win-x64` | ✅ Compresses | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `win-arm64` | ✅ Compresses | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `win-x86` | ✅ Compresses | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |

### Linux (via Zig)

| Target | UPX Support | Cross-compilation Tool |
|--------|-------------|----------------------|
| `linux-x64` | ✅ Compresses | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `linux-arm64` | ✅ Compresses | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `linux-musl-x64` | ✅ Compresses | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |
| `linux-musl-arm64` | ✅ Compresses | [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS) |

### macOS (native)

| Target | UPX Support | Notes |
|--------|-------------|-------|
| `osx-arm64` | ⏭️ Skipped | Auto-skipped due to macOS security |
| `osx-x64` | ⏭️ Skipped | Auto-skipped due to macOS security |

## How It Works

This package hooks into the Native AOT build process and automatically compresses the compiled binary:

**Build Flow:**

```
.NET AOT Compiler (macOS) → Native Binary → UPX Compression → Compressed Executable
                             (.exe/.elf)      (macOS tool)      (for target OS)
```

**Technical Details:**

1. **MSBuild Target**: Runs `AfterTargets="LinkNative"` to compress the binary immediately after linking
2. **Platform Detection**: Identifies target OS from RuntimeIdentifier (win-*/linux-*/osx-*)
3. **Smart Compression**: 
   - For Windows/Linux targets: runs UPX with `--best` flag
   - For macOS targets: skips compression and displays a message
4. **Host Tool**: Always uses macOS ARM64 UPX binary included in the package

**UPX Compression:**
- **Algorithm**: LZBA (default) or LZMA (optional)
- **Level**: `--best` (level 9) by default
- **Decompression**: Automatic at program launch, in-memory, typically unnoticeable

## Compression Results

For a Hello World program with size optimizations enabled:

| Target | Uncompressed | With `--best` UPX | Compression Ratio | Savings |
|--------|--------------|------------------|-------------------|---------|
| **Windows x64** | 1.05 MB | 483 KB | 44.9% | 55.1% |
| **Windows ARM64** | 1.01 MB | 455 KB | 45.0% | 55.0% |
| **Linux x64** | 1.30 MB | 520 KB | 40.0% | 60.0% |
| **Linux ARM64** | 1.25 MB | 500 KB | 40.0% | 60.0% |
| **macOS ARM64** | 1.20 MB | N/A (skipped) | - | - |

> 💡 **Tip**: Add `<PublishLzmaCompressed>true</PublishLzmaCompressed>` for even better compression (typically 5-10% smaller, but adds ~50-100ms to startup time).

## Deploying to Target Platforms

### Linux Deployment

.NET Native AOT binaries require the **ICU library** on the target system:

```bash
# Ubuntu/Debian
sudo apt-get install -y libicu-dev

# CentOS/RHEL/Fedora
sudo yum install -y icu

# Alpine Linux
apk add --no-cache icu-libs
```

**Docker Example:**

```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y libicu-dev
COPY YourApp /app/
CMD ["/app/YourApp"]
```

**Disable ICU (Optional):**

```xml
<PropertyGroup>
  <InvariantGlobalization>true</InvariantGlobalization>
</PropertyGroup>
```

### Windows Deployment

UPX-compressed Windows executables:
- ✅ Run on any Windows 10/11 system
- ✅ No additional runtime required (.NET Native AOT)
- ⚠️ May trigger false positives in some antivirus software (add to whitelist if needed)

**If Windows shows "compressed file" dialog:**
- This is a file association issue with WinRAR/7-Zip
- **Solution**: Right-click → Properties → Unblock (if shown)
- The `.exe` is directly executable, not a compressed archive

## Troubleshooting

### `upx: command not found`

Install UPX via Homebrew:

```bash
brew install upx
```

### Cross-compilation fails

Make sure you've installed the cross-compilation tools:
- For Windows: See [PublishAotCross.macOS Windows setup](https://github.com/interface95/PublishAotCross.macOS#for-windows-cross-compilation)
- For Linux: See [PublishAotCross.macOS Linux setup](https://github.com/interface95/PublishAotCross.macOS#for-linux-cross-compilation)

### UPX compression fails

Check that the package is properly installed:

```bash
# Verify UPX is available
upx --version

# Check package installation
dotnet list package | grep PublishAotCompressed.macOS
```

### "PublishAotCompressed.macOS can only be used on macOS"

This package only works on macOS. For other platforms:
- Original cross-platform version: [PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed)

## Related Projects

### Complete .NET Native AOT Cross-Compilation Ecosystem

| Project | Purpose | Platforms |
|---------|---------|-----------|
| **[PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS)** | Cross-compile from macOS | macOS → Windows/Linux |
| **PublishAotCompressed.macOS** (this) | UPX compression on macOS | Compresses Windows/Linux binaries |
| [PublishAotCross](https://github.com/agocke/PublishAotCross) | Cross-compile from Windows | Windows → Linux |
| [PublishAotCrossXWin](https://github.com/Windows10CE/PublishAotCrossXWin) | Cross-compile from Linux | Linux → Windows |
| [PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed) | Original UPX package | Multi-platform |

### Cross-Compilation + Compression Matrix

| Build Host | Target | Cross-compilation | UPX Compression |
|------------|--------|------------------|----------------|
| **macOS** | Windows | ✅ PublishAotCross.macOS | ✅ This package |
| **macOS** | Linux | ✅ PublishAotCross.macOS | ✅ This package |
| **macOS** | macOS | Native | ⏭️ Skipped (security) |
| Windows | Linux | ✅ PublishAotCross | ✅ PublishAotCompressed |
| Linux | Windows | ✅ PublishAotCrossXWin | ✅ PublishAotCompressed |

> 💡 **macOS users get the best of both worlds** - cross-compile to both Windows and Linux with automatic UPX compression from a single machine!

## Requirements

### Build Host
- **macOS** (Apple Silicon or Intel)
- **.NET 9.0 SDK** or later
- **Homebrew** (for installing tools)

### For Windows Cross-compilation
- **LLVM** (`lld-link` linker) - via `brew install lld`
- **Rust/Cargo** (for installing xwin) - via `brew install rust`
- **xwin** - via `cargo install xwin`
- **~1.5GB disk space** for Windows SDK

### For Linux Cross-compilation
- **Zig** (~200MB, includes everything) - via `brew install zig`

## Example Project

See the `test/` directory in this repository for a complete example.

## License

MIT License - see [LICENSE.TXT](LICENSE.TXT) for details.

## Credits

- Based on [PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed) by Michal Strehovsky
- UPX by [UPX Team](https://upx.github.io/)
- Designed to work with [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

- 🐛 **Issues**: [GitHub Issues](https://github.com/interface95/PublishAotCompressed.macOS/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/interface95/PublishAotCompressed.macOS/discussions)

---

**Made with ❤️ for the .NET Native AOT community on macOS**

