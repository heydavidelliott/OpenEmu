> [!IMPORTANT]
> **This is an unofficial beta branch** of OpenEmu by [@heydavidelliott](https://github.com/heydavidelliott). It is not an OpenEmu release. For official downloads, go to [openemu.org](https://openemu.org).

# Beta: wired Xbox controllers that macOS doesn't support

Many wired Xbox One / Series controllers don't work on macOS, including most third-party ones from PowerA, PDP and others. macOS has a driver for Xbox controllers, but it only accepts a few official Microsoft models. Every other controller that speaks the same protocol is ignored, so OpenEmu never sees it. The driver OpenEmu's wiki recommends, 360Controller, stopped working with macOS Big Sur.

This branch teaches OpenEmu to talk to these controllers directly over USB:

- **No driver, no security changes, no extra software.** Plug the controller in and OpenEmu finds it.
- **Full analog** sticks and triggers. Steering in Mario Kart 64 works as it should.
- **OpenEmu's existing Xbox presets** apply automatically on every system.
- **Hot-plugging:** unplug and replug the controller mid-game and it keeps working.
- **Other apps can still use the controller.** OpenEmu only takes it while a game is running or while you're on the Controls screen.

### Tested controllers

| Controller | USB ID | Status |
|---|---|---|
| PowerA Xbox Series X Wired Controller | `20d6:2062` | ✅ Works |

Other wired controllers that use the Xbox One protocol (PDP, Hori, Razer, PowerA Enhanced/Fusion and others) should work too, but they haven't been tested yet. If you try one, please [open an issue](https://github.com/heydavidelliott/OpenEmu/issues) with the controller's name, its USB ID (Apple menu → About This Mac → More Info → System Report → USB) and whether it worked.

Official Microsoft controllers that macOS already supports keep using Apple's driver. This branch leaves them alone.

### Known limitations

- **Quit Steam first.** While Steam is running, it holds the controller and OpenEmu can't use it. OpenEmu picks the controller up within a second of Steam quitting.
- **Wired USB only.** Wireless adapters aren't supported.
- **No rumble yet.** OpenEmu doesn't pass rumble from games to controllers. That's planned as a follow-up.

### Building the beta

You need a Mac with [Xcode](https://apps.apple.com/app/xcode/id497799835) 16.4 or later.

```sh
git clone -b gip-beta https://github.com/heydavidelliott/OpenEmu.git
cd OpenEmu
# One nested submodule uses an SSH URL; fetch everything over HTTPS instead.
git -c url."https://github.com/".insteadOf="git@github.com:" submodule update --init --recursive

xcodebuild -workspace OpenEmu.xcworkspace -scheme OpenEmu -configuration Release \
  -derivedDataPath build ARCHS=x86_64 ONLY_ACTIVE_ARCH=NO \
  CODE_SIGN_IDENTITY=- CODE_SIGN_STYLE=Manual DEVELOPMENT_TEAM=
open build/Build/Products/Release/OpenEmu.app
```

Notes:

- **The build is Intel (x86_64), on purpose.** It matches the official OpenEmu 2.4.1, and several emulator cores, including Mupen64Plus for N64, are Intel-only. It runs on Apple Silicon through Rosetta.
- **The beta shares your library, saves and settings with the official OpenEmu**, so don't run both at once. The library format is unchanged, so you can switch back at any time.
- Emulator cores are downloaded by OpenEmu as needed, the same way the official app does. Cores you already installed are reused.

### How it works

The controller support lives in [`OEGIPUSBDeviceHandler`](https://github.com/heydavidelliott/OpenEmu-SDK/blob/gip-beta/OpenEmuSystem/OEGIPUSBDeviceHandler.m) in the OpenEmu-SDK fork:

1. It finds controllers by their USB class, so no list of vendors is needed.
2. It opens the controller from user space and sends the same startup sequence as Linux's `xpad` driver.
3. It turns the controller's input reports into OpenEmu input events.

A USB controller can only be opened by one process at a time, and OpenEmu runs games in a separate helper process. So the app and the game hand the controller back and forth as needed.

The changes are being proposed upstream to OpenEmu. Until they're merged, this branch is the way to try them.

---

OpenEmu
=======

![alt text](http://openemu.org/img/intro-md.png "OpenEmu Screenshot")

OpenEmu is an open-source project whose purpose is to bring macOS game emulation into the realm of first-class citizenship. The project leverages modern macOS technologies, such as Cocoa, Metal, Core Animation, and other third-party libraries. One third-party library example is Sparkle, which is used for auto-updating. OpenEmu uses a modular architecture, allowing for game-engine plugins, allowing OpenEmu to support a host of different emulation engines and back ends while retaining the familiar macOS native front end.

Currently, OpenEmu can load the following game engines as plugins:

* Atari 2600 ([Stella](https://github.com/stella-emu/stella))
* Atari 5200 ([Atari800](https://github.com/atari800/atari800))
* Atari 7800 ([ProSystem](https://gitlab.com/jgemu/prosystem))
* Atari Lynx ([Mednafen](https://mednafen.github.io))
* ColecoVision ([CrabEmu](https://sourceforge.net/projects/crabemu/))
* Famicom Disk System ([Nestopia](https://gitlab.com/jgemu/nestopia))
* Game Boy / Game Boy Color ([Gambatte](https://gitlab.com/jgemu/gambatte))
* Game Boy Advance ([mGBA](https://github.com/mgba-emu/mgba))
* GameCube ([Dolphin](https://github.com/dolphin-emu/dolphin))
* Game Gear ([Genesis Plus](https://github.com/ekeeke/Genesis-Plus-GX))
* Intellivision ([Bliss](https://github.com/jeremiah-sypult/BlissEmu))
* NeoGeo Pocket ([Mednafen](https://mednafen.github.io))
* Nintendo (NES) / Famicom ([FCEUX](https://github.com/TASEmulators/fceux), [Nestopia](https://gitlab.com/jgemu/nestopia))
* Nintendo 64 ([Mupen64Plus](https://github.com/mupen64plus))
* Nintendo DS ([DeSmuME](https://github.com/TASEmulators/desmume))
* Odyssey² / Videopac+ ([O2EM](https://sourceforge.net/projects/o2em/))
* PC-FX ([Mednafen](https://mednafen.github.io))
* SG-1000 ([Genesis Plus](https://github.com/ekeeke/Genesis-Plus-GX))
* Sega 32X ([picodrive](https://github.com/notaz/picodrive))
* Sega CD / Mega CD ([Genesis Plus](https://github.com/ekeeke/Genesis-Plus-GX))
* Sega Genesis / Mega Drive ([Genesis Plus](https://github.com/ekeeke/Genesis-Plus-GX))
* Sega Master System ([Genesis Plus](https://github.com/ekeeke/Genesis-Plus-GX))
* Sega Saturn ([Mednafen](https://mednafen.github.io))
* Sony PSP ([PPSSPP](https://github.com/hrydgard/ppsspp))
* Sony PlayStation ([Mednafen](https://mednafen.github.io))
* Super Nintendo (SNES) ([BSNES](https://github.com/bsnes-emu/bsnes), [Snes9x](https://github.com/snes9xgit/snes9x))
* TurboGrafx-16 / PC Engine ([Mednafen](https://mednafen.github.io))
* TurboGrafx-CD / PCE-CD ([Mednafen](https://mednafen.github.io))
* Vectrex ([VecXGL](https://github.com/james7780/VecXGL))
* Virtual Boy ([Mednafen](https://mednafen.github.io))
* WonderSwan ([Mednafen](https://mednafen.github.io))

Minimum Requirements
--------------------

macOS Mojave 10.14.4

Building the default branch requires Xcode 14.3 and macOS Ventura.
