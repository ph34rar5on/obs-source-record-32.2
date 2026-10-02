# Source Record filter for OBS Studio

Plugin for OBS Studio to make sources available to record via a filter

# Download

# Build

Two methods are supported. Both were verified against OBS Studio 32.2.2.

Note: `-DBUILD_OUT_OF_TREE=On` is **not** needed. `CMakeLists.txt` detects a
stand-alone build automatically, because `CMAKE_PROJECT_NAME` is not
`obs-studio`.

## 1. In-tree build

Build OBS Studio: https://obsproject.com/wiki/Install-Instructions

1. Check out this repository to `plugins/source-record`
2. Add `add_subdirectory(source-record)` to `plugins/CMakeLists.txt`
3. Rebuild OBS Studio

If you only want the plugin (and not a full `obs64.exe`), you can skip most of
the OBS build. OBS fetches its own dependencies on first configure:

```
cmake -S <obs-studio> -B <build> -G "Visual Studio 18 2026" -A x64 ^
      -DENABLE_FRONTEND=OFF -DENABLE_PLUGINS=OFF ^
      -DENABLE_SCRIPTING=OFF -DENABLE_BROWSER=OFF
cmake --build <build> --config RelWithDebInfo -t obs-frontend-api source-record
```

The result is `<build>/plugins/source-record/RelWithDebInfo/source-record.dll`.

To install it into an existing OBS install, copy it next to your other 64-bit
plugin DLLs and copy `data/locale` to `data/obs-plugins/source-record/locale`:

```
copy <build>/plugins/source-record/RelWithDebInfo/source-record.dll ^
      "%ProgramFiles%\obs-studio\obs-plugins\64bit\"
xcopy /E /I data\locale ^
      "%ProgramFiles%\obs-studio\data\obs-plugins\source-record\locale"
```

## 2. Stand-alone build

Needs an OBS install that includes the development files. Those ship with the
`Development` CMake component — install them with:

```
cmake --install <build> --config RelWithDebInfo --prefix <prefix> --component Development
```

The plugin also needs the matching obs-deps on `CMAKE_PREFIX_PATH`, because
`libobsConfig.cmake` does `find_dependency(SIMDe)`.

**Linux**

```
cmake -S . -B build -DCMAKE_PREFIX_PATH="<prefix>;<obs-deps>"
cmake --build build
```

**Windows**

`CMAKE_PREFIX_PATH` needs *both* the OBS development prefix and the obs-deps
prefix, and `CMAKE_SYSTEM_VERSION` must match your installed Windows SDK:

```
cmake -S . -B build -G "Visual Studio 18 2026" -A x64 ^
      -DCMAKE_SYSTEM_VERSION=10.0.26100.0 ^
      -DCMAKE_PREFIX_PATH="<prefix>;<obs-deps>"
cmake --build build --config RelWithDebInfo
```

The DLL is written to `build/RelWithDebInfo/` and is mirrored into
`build/rundir/RelWithDebInfo/obs-plugins/64bit/`.
