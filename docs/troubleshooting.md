# Troubleshooting

This page contains known issues that may occur when running Counter-Strike 1.6 with Xash3D FWGS and CS16Client on Apple Silicon Macs.

For a normal installation, start with the steps in the main [README](../README.md).

---

## Game crashes when entering Player Name

On the first launch, the game may crash when entering your **Player Name** in the multiplayer menu.

Try launching the game once with console debugging enabled:

```bash
cd ~/CS16
./xash3d -game cstrike -condebug
```

Enter your player name and continue.

Afterwards, exit the game and launch normally:

```bash
./xash3d -game cstrike \
  +hud_scale 1.5 \
  +con_fontscale 2 \
  +hud_fontscale 1
```

If the game now starts normally, no further action is required.

The `-condebug` option enables console logging and can also be useful when investigating other crashes or runtime problems.

---

## `Can't initialize any renderer`

If Xash3D starts but reports:

```text
Can't initialize any renderer. Check your video drivers!
```

make sure the renderer libraries exist in the main `~/CS16` directory:

```text
libref_gl.dylib
libref_gles1.dylib
libref_gles2.dylib
libref_gles3compat.dylib
libref_soft.dylib
```

If they are missing, rebuild Xash3D with:

```bash
cd ~/cs16-macos-build/xash3d-fwgs

./waf configure \
  --build-type=release \
  --sdl-use-pkgconfig \
  --enable-gles1 \
  --enable-gles2 \
  --enable-gles3compat \
  --enable-soft

./waf build -j$(sysctl -n hw.ncpu)
```

Then copy the renderer libraries again:

```bash
cd ~/CS16

cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gl.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gles1.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gles2.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gles3compat.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/soft/libref_soft.dylib .
```

### Note about `--enable-all-renderers`

On some macOS configurations, enabling all Xash3D renderers may cause the `gl4es` component to fail during compilation.

If you see errors related to:

```text
gl4es
aliases are not supported on darwin
```

use the renderer configuration shown above instead of `--enable-all-renderers`.

---

## `native object "MenuFactory" is unavailable`

If the game starts but reports:

```text
native object "MenuFactory" is unavailable
```

make sure the CS16Client menu library exists in both locations:

```text
~/CS16/cstrike/menu_arm64.dylib
~/CS16/cstrike/cl_dlls/menu_arm64.dylib
```

If necessary, copy it again:

```bash
cp ~/cs16-macos-build/cs16-client/build/3rdparty/mainui_cpp/menu_arm64.dylib \
   ~/CS16/cstrike/

cp ~/cs16-macos-build/cs16-client/build/3rdparty/mainui_cpp/menu_arm64.dylib \
   ~/CS16/cstrike/cl_dlls/
```

---

## `Could not load model maps/...bsp from disk`

If you see an error such as:

```text
Could not load model maps/...bsp from disk
```

the required Counter-Strike game data is probably missing from `~/CS16/cstrike`.

First verify that the Steam files exist:

```bash
ls ~/cs16-steam/cstrike
```

If necessary, re-download/validate the game files with SteamCMD:

```text
app_update 10 validate
```

Then copy the `cstrike` directory to the Xash3D installation again:

```bash
cp -R ~/cs16-steam/cstrike ~/CS16/
```

If `~/CS16/cstrike` already exists and you want to replace it completely, remove the old directory first:

```bash
rm -rf ~/CS16/cstrike
cp -R ~/cs16-steam/cstrike ~/CS16/
```

After replacing the directory, remember to copy the CS16Client libraries from the main README again.

---

## CMake fails with C++ standard errors

If the CS16Client build fails with errors similar to:

```text
'auto' not allowed in function return type
```

make sure the project is configured with C++11:

```bash
cd ~/cs16-macos-build/cs16-client

cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_STANDARD=11 \
  -DCMAKE_CXX_STANDARD_REQUIRED=ON
```

Then rebuild:

```bash
cmake --build build --config Release
```

---

## SteamCMD installs the files in an unexpected location

Use an **absolute path** with `force_install_dir`:

```text
force_install_dir /Users/YOUR_USERNAME/cs16-steam
```

rather than:

```text
force_install_dir ~/cs16-steam
```

After running SteamCMD, verify that the game files are actually located at:

```text
~/cs16-steam/
├── cstrike/
├── valve/
└── ...
```

If the files were installed somewhere else, move or copy them to the expected location before continuing.

---

## `./waf` does not exist in CS16Client

Xash3D uses Waf, but CS16Client uses **CMake**.

For CS16Client, use:

```bash
cmake -S . -B build
```

and:

```bash
cmake --build build --config Release
```

Do not try to run `./waf` from the CS16Client directory.

---

## Checking the architecture

The whole point of this setup is to run natively on Apple Silicon.

You can verify that your Mac is ARM64 with:

```bash
uname -m
```

Expected output:

```text
arm64
```

You can also inspect individual binaries with:

```bash
file ~/CS16/xash3d
```

and:

```bash
file ~/CS16/cstrike/cl_dlls/client_arm64.dylib
```

The binaries should report an ARM64 architecture.

---

## Collecting debug information

If the game crashes or fails to start and the cause is unclear, launch it with:

```bash
cd ~/CS16
./xash3d -game cstrike -condebug
```

This enables console logging.

After reproducing the problem, check the generated console log in the game directory.

When reporting an issue, include:

- macOS version
- Mac model / Apple Silicon generation
- Xash3D FWGS commit/version
- CS16Client commit/version
- the exact command used to launch the game
- the relevant part of the console log

This makes it much easier to determine whether the issue is related to the engine, client, game assets, renderer, or macOS environment.

---

## Still stuck?

Make sure the installation follows the main [README](../README.md) from start to finish.

The most important things to verify are:

1. Your Steam installation contains the original `cstrike` and `valve` directories.
2. Xash3D was built successfully for ARM64.
3. CS16Client was built successfully for ARM64.
4. The required `.dylib` files were copied into the correct directories.
5. The renderer libraries are present in `~/CS16`.
6. The game is launched from the `~/CS16` directory.

If all of the above are correct, the game should be able to run natively on Apple Silicon.