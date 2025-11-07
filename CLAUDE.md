# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

### Prerequisites
- OpenJDK 11
- Android SDK (configured in `local.properties`)
- CMake
- Golang

### Setup
1. Update submodules:
   ```bash
   git submodule update --init --recursive
   ```

2. Create `local.properties` in project root:
   ```properties
   sdk.dir=/path/to/android-sdk
   ```

3. (Optional) Custom app package name in `local.properties`:
   ```properties
   custom.application.id=com.my.compile.clash
   remove.suffix=true
   ```

4. Create `signing.properties` for release builds:
   ```properties
   keystore.path=/path/to/keystore/file
   keystore.password=<key store password>
   key.alias=<key alias>
   key.password=<key password>
   ```

### Build Variants
- Debug builds:
  ```bash
  ./gradlew app:assembleDebug
  ./gradlew app:assembleAlphaDebug
  ./gradlew app:assembleMetaDebug
  ```
- Release builds:
  ```bash
  ./gradlew app:assembleRelease
  ./gradlew app:assembleAlphaRelease
  ./gradlew app:assembleMetaRelease
  ```

### Testing
- Run Go tests in core module:
  ```bash
  cd core/src/foss/golang/clash/test
  make test
  ```
- No Android instrumented tests are configured

## Architecture Overview

### Project Structure
- `:app` - Main Android application module
- `:core` - Core functionality with native C/Go integration
- `:service` - Service components (background services)
- `:design` - UI design components and themes
- `:common` - Common utilities and shared code
- `:hideapi` - Hide API library for accessibility service hiding

### Native Integration Architecture
The project uses a three-layer architecture:

1. **Java/Kotlin Layer** (`app` and other modules):
   - High-level UI and application logic
   - Calls through `Clash` singleton object
   - Uses `Bridge.kt` for native method declarations

2. **JNI Bridge Layer** (`core` module):
   - `main.c` - JNI entry points implementing Java native methods
   - `jni_helper.c/h` - JNI utilities (string conversion, exception handling)
   - `bridge_helper.c/h` - Bridge-specific helpers
   - Builds `libbridge.so` native library

3. **Go Core Layer** (`core/src/foss/golang/clash/`):
   - Clash.Meta kernel implementation
   - Compiled as `libclash.so`
   - Exports C functions to JNI layer via `bridge.h`

### Build Process
1. Go build: Compiles `libclash.so` with specific tags
2. CMake build: Compiles `libbridge.so` linking with `libclash.so`
3. Gradle: Combines everything into AAR and APK

### Key Components
- `Clash.kt` - Main singleton for Clash operations
- `Bridge.kt` - Native method declarations
- `TunInterface.kt` - TUN socket interface
- ExternalControlActivity - Service control via intents

### Product Flavors
- `alpha` - Default variant with Alpha suffix
- `meta` - Meta variant with Meta suffix
- Both flavors build the same core but with different application IDs and branding

### Automatic Features
- Geo database files are downloaded at build time
- APKs are split by architecture (arm64, armeabi, x86, x86_64)
- Git branch and commit hash are embedded in native library
- Automatic dependency updates via GitHub Actions

### Important Notes
- The Go kernel is pulled from MetaCubeX/Clash.Meta repository
- Version information is automatically generated from git
- Use Android Studio code style (File → Settings → Editor → Code Style → C/C++ and Kotlin → Scheme → Project)
- All contributions are under GPLv3 license