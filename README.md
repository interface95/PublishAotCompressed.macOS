# PublishAotCompressed.macOS

MSBuild targets to automatically compress Native AOT binaries with UPX on macOS. Designed to work seamlessly with cross-compilation tools like [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS).

## Features

- ✅ **Automatic UPX compression** after Native AOT compilation
- ✅ **Cross-compilation support** - compress binaries for Windows and Linux from macOS
- ✅ **macOS-only** - streamlined for macOS development workflow
- ✅ **60%+ size reduction** typical compression rates
- ✅ **Optional LZMA** for even better compression

## Prerequisites

- **macOS** (Apple Silicon or Intel)
- **.NET 9.0 SDK** or later
- **UPX** installed via Homebrew:
  ```bash
  brew install upx
  ```

## Quick Start

### 1. Install Cross-Compilation Tools

For cross-compiling to Windows/Linux, install [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS):

```xml
<PackageReference Include="PublishAotCross.macOS" Version="1.0.2-preview" />
<PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-dev" />
```

### 2. Configure Your Project

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
    <PackageReference Include="PublishAotCross.macOS" Version="1.0.2-preview" />
    <PackageReference Include="PublishAotCompressed.macOS" Version="1.0.0-dev" />
  </ItemGroup>
</Project>
```

### 3. Publish with Compression

#### For Windows (via lld-link + xwin):

```bash
# Make sure lld-link is in PATH
export PATH="$(brew --prefix lld)/bin:$PATH"

# Publish and compress
dotnet publish -r win-x64 -c Release
dotnet publish -r win-arm64 -c Release
```

#### For Linux (via Zig):

```bash
# Publish and compress
dotnet publish -r linux-x64 -c Release /p:StripSymbols=false
dotnet publish -r linux-arm64 -c Release /p:StripSymbols=false

# For Alpine Linux (musl)
dotnet publish -r linux-musl-x64 -c Release /p:StripSymbols=false
dotnet publish -r linux-musl-arm64 -c Release /p:StripSymbols=false
```

#### For macOS (native):

```bash
dotnet publish -r osx-arm64 -c Release
dotnet publish -r osx-x64 -c Release
```

> ✅ **Note**: macOS targets automatically skip UPX compression. The package will display a message and produce uncompressed binaries that run normally. This is by design since UPX-compressed macOS binaries cannot run due to security restrictions.

## Configuration

### Compression Settings

You can use either the full property name or the short alias:

```xml
<PropertyGroup>
  <!-- Short property name (recommended) -->
  <Upx>false</Upx>
  
  <!-- Or use the full property name -->
  <PublishAotCompressed>false</PublishAotCompressed>
  
  <!-- Use best compression (default: true) -->
  <CompressBest>true</CompressBest>
  
  <!-- Use LZMA for maximum compression (slower startup) -->
  <PublishLzmaCompressed>true</PublishLzmaCompressed>
</PropertyGroup>
```

**Command line usage:**

```bash
# Enable compression (short form) ✅ Recommended
dotnet publish -r win-x64 -c Release /p:Upx=true

# Disable compression (short form) ✅ Recommended  
dotnet publish -r win-x64 -c Release /p:Upx=false

# Or use the full property name
dotnet publish -r win-x64 -c Release /p:PublishAotCompressed=false
```

### Typical Results

For a Hello World program with size optimizations enabled:

| Target | Uncompressed | Compressed | Ratio |
|--------|--------------|------------|-------|
| Windows x64 | ~1.2 MB | ~500 KB | 58% |
| Linux x64 | ~1.3 MB | ~520 KB | 60% |
| macOS ARM64 | ~1.2 MB | N/A (skipped) | - |

## How It Works

This package hooks into the Native AOT build process (`AfterTargets="LinkNative"`):

1. **Detects platform**: Identifies target OS (Windows/Linux/macOS) from RuntimeIdentifier
2. **Selects UPX**: Uses macOS ARM64 UPX tool from the package
3. **Compresses binary**: Runs UPX with appropriate flags for the target platform
4. **Validates**: Ensures running on macOS (cross-compilation host)

**Build Flow:**

```
.NET AOT Compiler → Native Binary → UPX Compression → Compressed Executable
   (on macOS)         (.exe/.elf)      (macOS tool)      (for target OS)
```

## Cross-Compilation Matrix

| Source (Build Host) | Target Platform | UPX Support | Tool Required |
|---------------------|----------------|-------------|---------------|
| macOS ARM64 | win-x64/arm64 | ✅ Compresses | PublishAotCross.macOS |
| macOS ARM64 | linux-x64/arm64 | ✅ Compresses | PublishAotCross.macOS |
| macOS ARM64 | osx-arm64/x64 | ⏭️ Skipped (auto) | Native AOT |

## Important Notes

### macOS Target Binaries

This package **automatically skips UPX compression** for macOS targets (osx-arm64, osx-x64) because:
- UPX-compressed macOS binaries cannot run due to System Integrity Protection (SIP)
- Code signing is incompatible with UPX compression
- Gatekeeper blocks execution of compressed binaries

When you build for macOS, you'll see this message:
```
Skipping UPX compression for macOS target (osx-arm64). This package is designed 
for cross-compilation to Windows/Linux only.
```

**The produced macOS binary will be uncompressed and run normally.** No action needed!

### Linux Deployment

Native AOT Linux binaries require **ICU library** on target system:

```bash
# Ubuntu/Debian
sudo apt-get install -y libicu-dev

# Alpine Linux
apk add --no-cache icu-libs
```

Or disable ICU dependency:
```xml
<InvariantGlobalization>true</InvariantGlobalization>
```

## Troubleshooting

### UPX not found

```bash
brew install upx
```

### "PublishAotCompressed.macOS can only be used on macOS"

This package only works on macOS. For other platforms, use:
- [PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed) (original, multi-platform)

### Cross-compilation fails

Make sure you've installed the cross-compilation tools:
- For Windows: See [PublishAotCross.macOS Windows setup](https://github.com/interface95/PublishAotCross.macOS#for-windows-cross-compilation)
- For Linux: See [PublishAotCross.macOS Linux setup](https://github.com/interface95/PublishAotCross.macOS#for-linux-cross-compilation)

## Example Project

See the `test/` directory for a minimal example.

## Related Projects

- **[PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS)** - Cross-compilation from macOS to Windows/Linux
- **[PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed)** - Original multi-platform UPX compression package

## License

MIT License - see LICENSE.TXT for details.

## Credits

- Based on [PublishAotCompressed](https://github.com/MichalStrehovsky/PublishAotCompressed) by Michal Strehovsky
- UPX by [UPX Team](https://upx.github.io/)
- Designed to work with [PublishAotCross.macOS](https://github.com/interface95/PublishAotCross.macOS)
