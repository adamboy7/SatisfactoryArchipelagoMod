# SatisfactoryArchipelagoMod

Archipelago connection mod for [Satisfactory](https://www.satisfactorygame.com/).
Experience Satisfactory in a whole new way by randomizing your game progression, potentially bringing completely separate video games into the mix too!

Learn more about Archipelago [on the Archipelago website](https://archipelago.gg/).

Learn more about this mod by reading [the modpage on ficsit.app](https://ficsit.app/mod/Archipelago)
or the [Satisfactory Archipelago info page](https://archipelago.gg/games/Satisfactory/info/en).

## Playing the Game

For users new to Archipelago,
follow the setup directions on [ficsit.app](https://ficsit.app/mod/Archipelago).

For experienced users, or those looking for more information on how Archipelago changes Satisfactory,
and how to configure the mod for multiplayer,
see:

- [Satisfactory Archipelago info page](https://archipelago.gg/games/Satisfactory/info/en)
- [Satisfactory Archipelago setup guide](https://archipelago.gg/tutorial/Satisfactory/setup_en)

For working with versions of the Satisfactory apworld that haven't been merged into Archipelago core yet,
you can find the documentation on Jarno's fork of Archipelago, which is used for development.
Note that any Archipelago internal links will be broken there because you're viewing markdown files and not the deployed site.
You may also need to switch branches to see the version of the documentation you care about.

- [Jarno's Archipelago repository: en_Satisfactory.md](https://github.com/Jarno458/Archipelago/blob/Satisfactory/worlds/satisfactory/docs/en_Satisfactory.md)
- [Jarno Archipelago repository: setup_en.md](https://github.com/Jarno458/Archipelago/blob/Satisfactory/worlds/satisfactory/docs/setup_en.md)


## Get Support

Archipelago Discord: <https://discord.gg/8Z65BR2>

Satisfactory Modding Discord: <https://discord.ficsit.app>

## Development Setup

You should only need to follow the directions below if you want to contribute to developing the mod.

### Prerequisites

### Satisfactory Modding Development Environment

[Follow this setup guide](https://docs.ficsit.app/satisfactory-modding/latest/Development/BeginnersGuide/index.html)

You will need [this tutorial](https://docs.ficsit.app/satisfactory-modding/latest/Development/BeginnersGuide/ImportingAnotherMod.html) later to import mods' source to your project.

### Mod Dependencies

This mod depends on a few other Satisfactory mods.
You will need to import them into your project using the tutorial above before this mod can compile.

- [ContentLib](https://github.com/Nogg-aholic/ContentLib)
- [MAM Enhancer](https://github.com/Nogg-aholic/MAMTips)
- [Free Samples](https://github.com/budak7273/FreeSamples)
- [Fix Client Resource Sink Points](https://github.com/budak7273/FixClientResourceSinkPoints)
- [Hover Pack Fuse Reminder](https://github.com/budak7273/HoverpackFuseReminder)

### APCpp

This mod uses a specific branch of [APCpp](https://github.com/Jarno458/APCpp/tree/Satisfactory) to communicate with the AP server.

To update the used APCpp version:

- clone the [branch](https://github.com/Jarno458/APCpp/tree/Satisfactory) of APCpp
- Create a folder `build`
- Enter the folder
- `cmake .. -DWIN32=1`
- `cmake --build . --config Release`

1. Copy all `*.lib` over from `APCpp\build\` to `Archipelago\Source\APCpp\lib\Win64` and subdirectories
2. Copy all `*.h` over from `APCpp` to `Archipelago\Source\APCpp\inc`

#### Linux (dedicated server)

The Linux libs must be built with the same toolchain Unreal uses (clang 18 from the `v25_clang-18.1.0-rockylinux8` toolchain for UE 5.6)
and against that toolchain's libc++, not the system libstdc++, otherwise the server module will fail to link or crash.
These steps were done on Linux / WSL, using the Linux-hosted toolchain from
`https://cdn.unrealengine.com/Toolchain_Linux/native-linux-v25_clang-18.1.0-rockylinux8.tar.gz`,
which also contains the libc++ headers (`include/c++/v1`) UE 5.6 compiles against.

 * clone the [branch](https://github.com/Jarno458/APCpp/tree/Satisfactory) of APCpp with `--recurse-submodules`
 * The headers in `Source/APCpp/inc` come from an APCpp version that is not published yet.
   The current Linux libs were built from commit `bdd786f` with these headers copied in and the
   matching changes made: `AP_Send`, `AP_SetPackageReceivedCallback` (forwards `Retrieved`/`SetReply` packets),
   `AP_GetAllPlayers` returning `AP_NetworkPlayer`, `AP_QueryGiftBoxes` keyed by team and slot,
   and `AP_DataType::UnsignedInt64` removed
 * IXWebSocket's mbedTLS backend only loads system CA certificates on Windows;
   patch `SocketMbedTLS::loadSystemCertificates` in `IXWebSocket/ixwebsocket/IXSocketMbedTLS.cpp`
   to load a Linux CA bundle (e.g. `/etc/ssl/certs/ca-certificates.crt`), otherwise `wss://` fails and APCpp falls back to `ws://`
 * Create a CMake toolchain file `ue-linux-toolchain.cmake`:
   ```cmake
   set(CMAKE_SYSTEM_NAME Linux)
   set(CMAKE_SYSTEM_PROCESSOR x86_64)
   set(UE_TC_ROOT "/path/to/v25_clang-18.1.0-rockylinux8/x86_64-unknown-linux-gnu")
   set(UE_LIBCXX_INC "${UE_TC_ROOT}/include")
   set(CMAKE_SYSROOT "${UE_TC_ROOT}")
   set(CMAKE_C_COMPILER "${UE_TC_ROOT}/bin/clang")
   set(CMAKE_CXX_COMPILER "${UE_TC_ROOT}/bin/clang++")
   set(CMAKE_AR "${UE_TC_ROOT}/bin/llvm-ar" CACHE FILEPATH "")
   set(CMAKE_RANLIB "${UE_TC_ROOT}/bin/llvm-ranlib" CACHE FILEPATH "")
   set(CMAKE_C_COMPILER_TARGET x86_64-unknown-linux-gnu)
   set(CMAKE_CXX_COMPILER_TARGET x86_64-unknown-linux-gnu)
   set(_common "-fPIC -fvisibility=hidden -ffunction-sections -fdata-sections")
   set(CMAKE_C_FLAGS_INIT "${_common}")
   set(CMAKE_CXX_FLAGS_INIT "${_common} -fvisibility-inlines-hidden -nostdinc++ -isystem ${UE_LIBCXX_INC} -isystem ${UE_LIBCXX_INC}/c++/v1")
   set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
   set(CMAKE_FIND_ROOT_PATH "${UE_TC_ROOT}")
   set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
   set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
   set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
   set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
   ```
 * Create a folder `build` and enter it
 * `cmake .. -DCMAKE_TOOLCHAIN_FILE=/path/to/ue-linux-toolchain.cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_DISABLE_FIND_PACKAGE_OpenSSL=ON -DCMAKE_DISABLE_FIND_PACKAGE_PkgConfig=ON -DBUNDLED_JSONCPP=ON -DUSE_MBED_TLS=ON -DMBEDTLS_FATAL_WARNINGS=OFF`
 * `cmake --build . -j`

1. Copy `libAPCpp.a`, `IXWebSocket/libixwebsocket.a`, `jsoncpp/src/lib_json/libjsoncpp.a` and `_deps/zlib-build/libz.a` to `Archipelago\Source\APCpp\lib\Linux`
2. Copy all `*.a` from `mbedtls_bin/library` and `mbedtls_bin/3rdparty/*` to `Archipelago\Source\APCpp\lib\Linux\mbedtls`

### Archipelago Server

You need to have a running Archipelago server to test the mod in-game.
You can self host one from source by following the directions
[in the Archipelago repo](https://github.com/ArchipelagoMW/Archipelago/blob/main/docs/running%20from%20source.md).
