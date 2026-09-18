# Counter-Strike 1.6 on Apple Silicon Mac

Run **Counter-Strike 1.6 natively on Apple Silicon Macs (ARM64)** using [Xash3D FWGS](https://github.com/FWGS/xash3d-fwgs) and [CS16Client](https://github.com/Velaron/cs16-client).

This guide builds the engine and client from source and uses the original game files from your Steam installation.

> **Legal note:** You must own Counter-Strike 1.6. This guide does not distribute any original Counter-Strike game assets.

## Requirements

- Apple Silicon Mac (M1/M2/M3/M4 or later)
- macOS 13 or later
- Counter-Strike 1.6 on Steam
- Homebrew
- Git
- CMake
- SDL2
- SteamCMD

Install the required tools:

```bash
brew install git cmake sdl2 steamcmd
```

---

## 1. Get the Counter-Strike 1.6 game files

Create a directory for the Steam game files:

```bash
mkdir -p ~/cs16-steam
```

Start SteamCMD:

```bash
steamcmd
```

Inside SteamCMD, run:

```text
force_install_dir /Users/YOUR_USERNAME/cs16-steam
login YOUR_STEAM_USERNAME
app_update 10 validate
quit
```

Replace `YOUR_USERNAME` and `YOUR_STEAM_USERNAME` with your own values.

`10` is the Steam App ID for Counter-Strike 1.6.

After installation, the directory should contain the original game data:

```text
~/cs16-steam/
├── cstrike/
├── valve/
└── ...
```

---

## 2. Download the source code

Create a build directory:

```bash
mkdir -p ~/cs16-macos-build
cd ~/cs16-macos-build
```

Clone Xash3D FWGS:

```bash
git clone --recursive https://github.com/FWGS/xash3d-fwgs.git
```

Clone CS16Client:

```bash
git clone --recursive https://github.com/Velaron/cs16-client.git
```

---

## 3. Build Xash3D FWGS

Enter the Xash3D directory:

```bash
cd ~/cs16-macos-build/xash3d-fwgs
```

Configure the build:

```bash
./waf configure \
  --build-type=release \
  --sdl-use-pkgconfig \
  --enable-gles1 \
  --enable-gles2 \
  --enable-gles3compat \
  --enable-soft
```

Build:

```bash
./waf build -j$(sysctl -n hw.ncpu)
```

The main runtime files will be generated here:

```text
build/game_launch/xash3d
build/engine/libxash.dylib
build/filesystem/filesystem_stdio.dylib
```

Renderer libraries are generated under:

```text
build/ref/gl/
build/ref/soft/
```

---

## 4. Build CS16Client

Enter the CS16Client directory:

```bash
cd ~/cs16-macos-build/cs16-client
```

Configure the project:

```bash
cmake -S . -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_STANDARD=11 \
  -DCMAKE_CXX_STANDARD_REQUIRED=ON
```

Build:

```bash
cmake --build build --config Release
```

The build should produce the following ARM64 libraries:

```text
build/cl_dll/client_arm64.dylib
build/3rdparty/ReGameDLL_CS/regamedll/cs_arm64.dylib
build/3rdparty/yapb/yapb_arm64.dylib
build/3rdparty/mainui_cpp/menu_arm64.dylib
```

---

## 5. Create the game directory

Create the final runtime directory:

```bash
mkdir -p ~/CS16
```

Copy the Xash3D runtime:

```bash
cp ~/cs16-macos-build/xash3d-fwgs/build/game_launch/xash3d ~/CS16/
cp ~/cs16-macos-build/xash3d-fwgs/build/engine/libxash.dylib ~/CS16/
cp ~/cs16-macos-build/xash3d-fwgs/build/filesystem/filesystem_stdio.dylib ~/CS16/
```

Copy the original Steam game data:

```bash
cp -R ~/cs16-steam/valve ~/CS16/
cp -R ~/cs16-steam/cstrike ~/CS16/
```

---

## 6. Install the CS16Client libraries

Create the required directories:

```bash
mkdir -p ~/CS16/cstrike/cl_dlls
mkdir -p ~/CS16/cstrike/dlls
```

Copy the client:

```bash
cp ~/cs16-macos-build/cs16-client/build/cl_dll/client_arm64.dylib \
   ~/CS16/cstrike/cl_dlls/
```

Copy ReGameDLL:

```bash
cp ~/cs16-macos-build/cs16-client/build/3rdparty/ReGameDLL_CS/regamedll/cs_arm64.dylib \
   ~/CS16/cstrike/dlls/
```

Copy YaPB:

```bash
cp ~/cs16-macos-build/cs16-client/build/3rdparty/yapb/yapb_arm64.dylib \
   ~/CS16/cstrike/dlls/
```

Copy the menu library:

```bash
cp ~/cs16-macos-build/cs16-client/build/3rdparty/mainui_cpp/menu_arm64.dylib \
   ~/CS16/cstrike/
```

The menu library also needs to be available in `cl_dlls`:

```bash
cp ~/cs16-macos-build/cs16-client/build/3rdparty/mainui_cpp/menu_arm64.dylib \
   ~/CS16/cstrike/cl_dlls/
```

---

## 7. Install the renderers

Copy the Xash3D renderer libraries into the main `CS16` directory:

```bash
cd ~/CS16

cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gl.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gles1.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gles2.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/gl/libref_gles3compat.dylib .
cp ~/cs16-macos-build/xash3d-fwgs/build/ref/soft/libref_soft.dylib .
```

The final directory should now look roughly like:

```text
~/CS16/
├── cstrike/
├── valve/
├── xash3d
├── libxash.dylib
├── filesystem_stdio.dylib
├── libref_gl.dylib
├── libref_gles1.dylib
├── libref_gles2.dylib
├── libref_gles3compat.dylib
└── libref_soft.dylib
```

---

## 8. Launch Counter-Strike 1.6

Run:

```bash
cd ~/CS16
./xash3d -game cstrike
```

If everything is installed correctly, Counter-Strike 1.6 should start using native ARM64 binaries.

### Recommended launch command

For better HUD and console readability on high-resolution Mac displays:

```bash
./xash3d -game cstrike \
  +hud_scale 1.5 \
  +con_fontscale 2 \
  +hud_fontscale 1
```

You can adjust these values to your preference.

---

## Troubleshooting

The normal installation above should be enough for a working setup.

If you run into one of the known issues, see:

**[Troubleshooting →](docs/troubleshooting.md)**

---

## References

- [Xash3D FWGS](https://github.com/FWGS/xash3d-fwgs)
- [CS16Client](https://github.com/Velaron/cs16-client)
- [cs16-macos](https://github.com/4xiomdev/cs16-macos)

## Credits

This project relies on the excellent work of the Xash3D FWGS and CS16Client projects.

This repository provides a build/setup guide for running Counter-Strike 1.6 on Apple Silicon Macs using your own legally obtained game files.