# UPX Binaries

This package contains UPX (Ultimate Packer for eXecutables) for macOS.

## Included Tools

- **macOS ARM64**: UPX 5.0.2 from Homebrew

## Source

UPX can be installed via Homebrew:
```bash
brew install upx
```

## Usage in PublishAotCompressed.macOS

The UPX binary in this package is used to compress Native AOT binaries for:
- Windows (win-x64, win-arm64, win-x86)
- Linux (linux-x64, linux-arm64, linux-musl-x64, linux-musl-arm64)
- macOS (osx-arm64, osx-x64) - **Note: Compressed macOS binaries may not run**

## Cross-Platform Support

UPX is a universal compressor that can compress binaries for different platforms:
- **Windows PE** executables (.exe)
- **Linux ELF** binaries
- **macOS Mach-O** binaries (with `--force-macos` flag)

This makes it perfect for cross-compilation scenarios where you build on macOS but target Windows or Linux.

## macOS Target Note

macOS binaries compressed with UPX can be built successfully but may not run due to macOS security restrictions:
- System Integrity Protection (SIP)
- Code signing requirements
- Hardened Runtime enforcement

**Recommendation**: Disable compression for macOS targets:
```bash
dotnet publish -r osx-arm64 -c Release /p:PublishAotCompressed=false
```

## License

UPX is licensed under the GNU General Public License v2.0 with exceptions.
See the COPYING file for details.

## More Information

- UPX Homepage: https://upx.github.io/
- UPX GitHub: https://github.com/upx/upx
