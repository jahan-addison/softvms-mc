# softvms

A fork of the Sega VMS emulator by Marcus Comstedt.

<h5 align="center"> Now compiles for Windows, Apple Silicon, and Linux platforms </h5>
<p align="center">
   <img src="/docs/screenshot.png" width="750px">
</p>

## Dependencies

- A C99 compiler (clang or gcc)
- CMake ≥ 3.16
- X11 — on macOS, install [XQuartz](https://www.xquartz.org):
  ```sh
  brew install --cask xquartz
  ```
- SDL (optional, for audio)

## Build

```sh
cmake -B build
cmake --build build
```

## Running on macOS (Apple Silicon)

XQuartz must be running before launching the emulator.

* Install XQuartz (run once):
   ```sh
   brew install --cask xquartz
   ```
* Reset shell for `DISPLAY` launch agent registered by XQuartz.
* Start XQuartz and run the emulator:
   ```sh
   open -a XQuartz
   ./build/vms
   ```

   If `DISPLAY` is still unset in your current session, set it manually:
   ```sh
   open -a XQuartz
   export DISPLAY=:0
   ./build/vms
   ```

## Usage

```
vms [-h] [-V] [-2] [-b bios] game.vms ...
```

## Resources

* VMS software: https://mc.pp.se/dc/sw.html
* VMS hardware: https://mc.pp.se/dc/hw.html
* Extras: https://www.deco.franken.de/myfiles/myfiles.html

