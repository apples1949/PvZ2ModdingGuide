# PvZ-2 MOD Technical Overview & Getting Started Guide

This documentation was created by TwinStar (aka smallpc - the creator of SPC-Util, Taiji, PvZTool, TwinKles | TwinStar)
Download TwinStar.Toolkit to start your PvZ2-Mod: https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document
Follow TwinStar.Toolkit update development: https://discord.com/invite/v7qvttSX8K

This document is a brief overview of PvZ-2 MOD making techniques, providing a rough guide for those who wish to get started with MOD making.

- [PvZ-2 MOD Technical Overview and Getting Started Guide](#pvz-2-mod-technical-brief-and-start-guide)
  - [Game Version](#Game-Version)
  - [Application Composition and Modification Routes](#Application-Composition-and-Modification-Routes)
  - [Modification Program](#Modification-Program)
    - [Overview](#Overview)
    - [Examples](#Examples)
  - [Resource Modification](#Resource-Modification)
    - [Resource file location](#Resource-file-location)
    - [Class of resource file](#Class-of-resource-file)
      - [RSB \& RSG](#rsb--rsg)
      - [RTON](#rton)
      - [PTX](#ptx)
      - [PAM](#pam)
      - [POPFX](#popfx)
      - [WEM](#wem)
      - [BNK](#bnk)
      - [TXT](#txt)
      - [XML](#xml)
      - [JSON](#json)

## Game-Version

According to the running platform, the game is divided into `Android` version and `iPhone` version.

Depending on the region, the game is divided into three versions as follows:

- `ROW = Rest Of World` ：International version, for most regions outside North America and China.

- `NA = North America` ：North American version, for North America, and `ROW` in the game experience is almost no difference, only for the different regions.

- `CHS = CHinese Simplified` ：Simplified Chinese version for China mainland, developed by PopCap Shanghai (the former operator, now disbanded) and TalkWeb (the current operator), which has a lot of different from the international version in terms of game experience.

Since there is almost no difference between `NA` and `ROW`, they are sometimes referred to as the international version, and this document will mainly refer to both `ROW` and `NA`versions, and the version the author mention in this document is `ROW` for Android, you can understand the same with `NA`.

## Application-Composition-and-Modification-Routes

For the `Android - ROW` version, the game application consists of the following parts:

- `<APK>` application installation package, which is essentially a special `ZIP` file.

  - `lib` native library directory, which contains `SO = Shared Object` files, which are dynamic library files under Linux that contain machine instructions that the CPU can execute directly.

    - `armeabi-v7a` Native library directory for 32-bit arm devices.

      - `libPVZ2.so` The compiled product of the game's main source code.

      - `...` Other dynamic library files.

    - `arm64-v8a` Native library directory for 64-bit arm devices, identical to `armeabi-v7a`.

  - `classes<n>.dex` Dalvik virtual machine executable that calls `libPVZ2.so` to run the game when it starts.

  - `...` Other miscellaneous files.

- `<Internal-Storage>` The device's internal storage, the actual path is `/data/user/<id>` 。

  - `<package-name>` Internal storage for the application, not a concern for the `ROW` version.

- `<External-Storage>` External storage for the device, the actual path is `/storage/emulated/<id>` 。

  - `Android`

    - `data/<package-name>` The external storage for the app.

      - `files/No_Backup` The game's archive directory (Additional: This place also have connects with PopCap's server, the game will check & download the new CDN when you launch the game with Internet Connection).

        - `locao_profile` Local configuration, which holds some game settings, essentially RTON files.

        - `pp.dat` User archive, essentially an RTON file, `pp = Player Profile` (Additional: With PvZ2 International version, this file can be located in Android/data but with the Chinese version for China mainland, it is located in data/data which requires you to root the device to access it).

        - `snapshot<id>.dat` A snapshot of the user archive, essentially an RTON file that should be deleted after each modification to the game archive.

        - `CDN.<major>.<minor>` CDN distribution for the current game version, where the files will overwrite the corresponding files in the `Packages` resource group in the packet at runtime.

        - `...` Other miscellaneous files.

    - `obb/<package-name>` The application's extended packet storage directory.

      - `main.<version-code>.<package-name>.obb` The game's extension package, which contains all resources used by the game such as images, animations, text, etc., is essentially an RSB file. (Additional: If you want to use PopCap RSB Unpack with TwinStar.Toolkit, you need to rename the extension name ".obb" to ".rsb" so the tool can detect it is the game RSB because ".obb" is Android natural extension name)

Depending on the modification target, the game's modification routes are divided into the following two categories:

- Modification program: Modify the main body program of the game (i.e. APK, especially the DEX and SO in it), which requires high technical requirements for the modifier and requires certain knowledge of software reversal, which can customize the operation logic of the game and can make any effect in theory.

- Resource modification: modify the main resource data of the game (mainly the extended packets and user archives), which requires low skills of the modifier and can be coded and decoded by the existing tools and programs, and can only be modified under the rules set by the program itself, with having a lot of certain limitations due to the hardcode.

## Modification-Program

### Overview

Program modification is the modification of the game's program behavior. Program modification allows you to customize the game's running logic entirely and theoretically make any effect you wish.

The essence of program modification is the reverse engineering of the program and the principle is as follows:

1. The source code in the software development stage is compiled when the software is distributed to generate code that can be executed directly by the CPU or interpreter.

   - Native languages such as `C` 、`C++`、etc... the source code is compiled into platform-specific CPU-oriented machine code

   - For languages such as `Java`, the source code is compiled to bytecode for a specific interpreter (`JVM`, etc.). 2.

2. The compilation products of the source code determine the execution logic of the program, and modifying these compilation products changes the execution logic of the program to make it behave the way the modifier wants it to behave (break hardcode).

3. The analysis and modification of these compilation products is called software inversion, which requires the modifier to have knowledge of x86/arm assembly, smail, etc.

> Here we need to correct a common fallacy: program modification is modifying the source code of a game.
>
> Some people do not have knowledge about software development and mistakenly consider the modification of the source code compilation product as the modification of the source code itself；\
> The source code is at development level rather than user level, only game operators can access and modify the source code, third parties can only steal the source code by hacking or imitate/recover the source code based on reverse analysis (high technical requirements and great workload), so far there is no third party can do one of the two, so no third party can modify the source code of the game, only the compiled the product is modified.

The main object of program modification is the DEX file and SO file in APK, where `libPVZ2.so` contains the main source code of the game. These files can be reverse-analyzed and modified with the help of IDA, dex2jar and other tools.

Reverse engineering itself is a complex area, this document does not explain more, interested parties can find information on the Internet to learn by themselves.

### Examples

- 0 sun mod & no cooldown

  By modifying `libPVZ2.so`, you can set the number of sun and cooldown time required to grow plants to zero.

  A traditional and simple way to do this is to remove the `Cost` and `PacketCooldown` string constants from so, so that the game does not get the sun count and cooldown data when deserializing RTON data into `RtObject` objects, and only uses the default value of `0`, thus giving the effect of 0 sun and no cooldown.

- Modify the game frame rate (Also known as FPS)

The screen refresh rate of PvZ-2 is controlled by DEX, where the program refreshes the screen every `33 ms`, resulting in a frame rate of `30 fps`, which can be modified to `17 ms` to achieve `60 fps` or any other `fps` number.

This is not recommended: firstly, the high refresh rate will be a burden to the device; secondly, many animations in the game are encoded at `30 fps` or `24 fps`, modifying the refresh rate does not change the actual frame rate of these animations, and the animation file must be framed to achieve high frame rate animations, which requires a lot of work and is not realistic.

- Modify character buffer size

  The international version of PvZ-2 has a limited character buffer for text rendering. Since the international version is only available for English, French, Spanish and other six languages, the preset character buffer is sufficient for the small number of characters in these six languages; however, for Chinese, a language with a very large number of characters, the preset character buffer is not enough to accommodate the rich variety of characters, which causes the Chinese MOD of PvZ-2 International Edition to be prone to character disappearance and even program crash when playing having a lot of problem.

  You can try to modify the game's character buffer to alleviate this problem, the relevant program code is located in a dummy function of the `Sexy::PrimeGlyphCache` class in `libPVZ2.so`. You can refer to my [personal column](https://www.bilibili.com/read/cv16512322?spm_id_from=333.999.0.0) on BiliBili, but the method described there only works for low-end devices (768, 384) and not for high-end devices (1536) and the language is Chinese so you should use a translator on the column.

- Unblock unfinished worlds from older versions

  In earlier versions of PvZ-2 International (around 1.4), the game implemented some of the Far-Future and Dark-Ages worlds encoded and resourced, but these two worlds were disabled in the program body when being delivered to the user, and the player could not retrieve them by modifying only the RSB packets.

  This restriction can be removed by modifying `libPVZ2.so`.

- Private server

  By providing packet capture and reverse of the program, you can analyze the rules of the game and server communication to build your own self-service.

- `Project Paradox`

  `Paradox` is a PvZ-2 MOD project, which uses the means of program modification to make some very interesting effects, the following is a brief description of the project modification principle:

  1.  Introduce the self-compiled `LawnMod.so` dynamic library file into the installation package (You can rename ".apk" to ".zip" to view the file inside).

  2.  Modify the `DEX` file to force the game loads `LawnMod.so` and calls the initialization modules inside it.

  3.  In the initialization modules of `LawnMod.so`, perform HOOK on many code segments in `libPVZ2.so` that have been loaded into memory to customize the default logic of the game to fits the modifier's wants.

      For example, this project HOOKs the snippet of `libPVZ2.so` about the amount of plant ids code, and uses a custom modules to evaluate instead of the original module, and first calls the original module in the custom module, and then defines an additional plant ids code, thus adding a custom plant ids code to the game, allowing the player to get and keep the custom plants of the MOD in the archive. The principle of the mod is nothing special.

  The modification principle of this MOD is not special, but the hardest part is to write the HOOK logic, which is a big challenge to the modifier requires the modder to do enough reverse analysis of `libPVZ2.so`.

## Resource-Modification

Resource modification means modifying the game's resource data, which refers to all the data files of the game except the program main body (hardcode), and in the case of PvZ-2, the resource data mainly refers to the expansion packets and the files in the archive directory.

Resource modification can only be done under the rules set by the program main body, which has a lot of certain limitations.

Thanks to the development of open technical of the PvZ-2 MOD, the threshold for resource modification is very low, and the modders only need basic computer knowledge to start their modifying. Modification programs can be found in [Fandom wiki](https://ernestoam.fandom.com/wiki/Plants_vs._Zombies_2_Hacking_Tools) , the one used in this document is [`TwinStar ToolKit`](https://github.com/twinkles-twinstar/TwinStar.ToolKit) ，please follow the [Documentation](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document) to install the toolkit (Which is written in Chinese).

The "tool" below refers to the `TwinStar ToolKit`, but you can also try other open tools for file processing.

It is recommend to use `TwinStar Toolkit` with more functions. If you need a tutorial video, here is the tutorial video by the author of `TwinStar Toolkit` on [Youtube](https://youtu.be/NokJrWGXOh4).

### Resource-file-location

Depending on the location of the resource files in the storage, they are divided into the following two categories:

- In-package resources: The vast majority of the game's resources are packaged in a single RSB packet file, which is stored in the application's OBB directory in the form of an application extension packet. Changes to the in-package resources require unpacking and repackaging of the RSB.

- Out-of-package resources: A few resources of the game are loosely stored in the game's archive directory, including user archive `pp.dat`, local configuration `local_profile`, unfinished level cache `saves`, CDN distribution `CDN.<major>. <minor>`, etc.

### Class-of-resource-file

The resource files used in PvZ-2 are of various types and are mostly PopCap's own private formats, which need to be decoded and re-encoded with the help of tools.

For example, `pp.dat` and `local_profile` files are RTON files without the `.rton` extension, and some RTON files in the CDN distribution directory have the `.json` extension with it.

#### RSB & RSG (RSGP)

In this document, the author mention ".rsb". The extension ".obb" in Android is actually ".rsb", so you can understand this game ".obb" is ".rsb".

`RSB = Resource Stream Bundle` and `RSG = Resource Stream Group` are the resource packet formats used by some PopCaps, including PvZ-2.

`TwinStar Toolkit` is a tool strictly asking for version number, so the version number of PvZ-2 RSBs & RSGs are 4 for both PvZ-2 International version & PvZ-2 Chinese version for China mainland.

RSG is a wrapper for resources, but RSG itself cannot describe the complete resource information and must be embedded in RSB; RSB is a wrapper for RSG, which describes the complete resource information and groups the resource files according to the resolution and regional differences, so that the game can selectively load the required resources according to the actual situation of the device.

In PvZ-2, there exists only a unique RSB file, which is the extended packet file of the application ``<External-Storage>/Android/obb/<package-name>/main.<version-code>. <package-name>.obb`.

In PvZ-2, the RSG does not exist as a single file, but is embedded in the RSB package.

The RSB holds the various resource files used by the game, and can be unpacked and packaged using tools that allow modders to remove resources or add custom resources as allowed locally by the program. The RSB can easily be damaged by the modders in some data that the current game does not read & this behavior has been repaired when you use the function called "RSB Repair" provided by `TwinStar Toolkit`.

#### RTON

`RTON = ReflecTion Object Notation` is the object notation format used by some PopCaps, including PvZ-2 & PvZ2C (You can found it in Packages).

RTON is the binary representation of `Sexy::RtObject` objects in the PopCap SexyFramework (the game development framework on which PvZ-2 is based); the process of encoding from RtObject objects to RTON is called serialization, and the process of decoding from RTON to RtObject objects is called deserialization.

RTON is a binary format, which is not suitable for human reading and modification, so it is generally not modified directly, but decoded into JSON with the help of modding tools, modified the textual JSON file, and then re-encoded into RTON using modding tools.

In PvZ-2 Chinese version, RTON files are encrypted with a layer of Rijndael

The tool supports encoding, decoding, encryption and decryption of RTON, where the keys required for encryption and decoding need to be found by the user from the game program files.

After getting the key, you can use `TwinStar Toolkit` to run `PopCap ReflecTion Object Notation Decrypt & Decode` and `PopCap ReflecTion Object Notation Encode & Encrypt` to encrypt the file inside.

```json
{
  "version": 1, // RTON version, always 1
  "objects": [
    // the RtObject objects recorded in this RTON file
    {
      // an RtObject object
      "uid": "1.2.33445566", // The Unique-ID of the object, which can be referenced by RTID(uid@rton_file_name)
      "aliases": ["XX"], //  alias of the object, which can be referenced by RTID(alias@rton_file_name), which target directly the rton file path you point to
      "objclass": "...", // The actual type of the object
      "objdata": {
        // The properties of the object, depending on the objclass, the properties inside the objdata are also different
        "property_1": true, // The boolean property
        "property_2": 2.4, // The number property
        "property_3": "a string value" // The string property
      }
    }
    // ...
  ]
}
```

The game's `libPVZ2.so` can be analyzed by the [PvZ2LibraryAnalyzer project](https://github.com/twinkles-twinstar/TwinStar.ToolKit.PvZ2LibraryAnalyzer) to parse out the RTON class property (objdata) writing specification, which will be useful for RTON modifications.

You can also find the 9.9.1 version of the RTON class property specification table from [Miscellaneous 分发](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document/releases/tag/Miscellaneous) .

#### PTX

`PTX = Popcap TeXture` is the image texture format used by some PopCaps, including PvZ-2.

Toolkit can decode PTX textures to PNG images or encode PNG images to PTX textures.

PTX generally only stores the image texture data, and does not contain the image size and texture format information, which is recorded in the RSB packet where the PTX is located. format.

The `format` field recorded in the RSB manifest file is the serial number value of the texture format type stored in the PTX, and its correspondence to the texture format type is shown in the following table:

| Device  | Version Number |   Texture Format   |
| :-----: | :------------: | :----------------: |
| iPhone  |       0        |     argb_8888      |
| Android |       0        |    rgba_8888_o     |
|   any   |       1        |     rgba_4444      |
|   any   |       2        |      rgb_565       |
|   any   |       3        |     rgba_5551      |
|   any   |       21       |  rgba_4444_tiled   |
|   any   |       22       |   rgb_565_tiled    |
|   any   |       23       |  rgba_5551_tiled   |
|   any   |       30       |    rgba_pvrtc4     |
|   any   |       31       |    rgba_pvrtc2     |
|   any   |       32       |      rgb_etc1      |
|   any   |       33       |      rgb_etc2      |
|   any   |       34       |     rgba_etc2      |
|   any   |       35       |      rgb_dxt1      |
|   any   |       36       |     rgba_dxt3      |
|   any   |       37       |     rgba_dxt5      |
|   any   |       38       |     rgb_atitc      |
|   any   |       39       |     rgba_atitc     |
|   ROW   |      147       |    rgb_etc1_a_8    |
|   CHS   |      147       | rgb_etc1_a_palette |
|   any   |      148       |   rgb_pvrtc4_a_8   |
| iPhone  |      149       |   argb_8888_a_8    |
| Android |      149       |  rgba_8888_o_a_8   |
|   CHS   |      150       | rgb_etc1_a_palette |

> Some of the texture formats are not actually used in PvZ-2 games, but were discovered by reverse analysis of `libPVZ2.so`.
>
> Some texture formats have the `_o` suffix, which means that the texture data is endian-order independent (data is encoded byte-by-byte), in OpenGL, libpng, RGBA_8888 textures are endian-order independent and the data is arranged byte-by-byte in the order R, G, B, A.

When decoding PTX to PNG using the tool, you need to provide the width, height, and texture format of the image, all of which can be found in the RSB packet manifest file, while when encoding PNG to PTX, you only need to provide the texture format.

Some texture formats require the image dimension (e.g. ETC requires a square image, width and height to the Nth power of two), and the tool will automatically fill the size to the required dimension when encoding and decoding PTX with the value provided by the user.

> For example, the following is the information of a PTX file described in the Android RSB manifest file:
>
> ```json
> "ATLASES/ALWAYSLOADED_1536_00.PTX": { // Path to the PTX file within the package in RSB
> 	"additional": {
> 		"type": "texture",
> 		"value": {
> 			"size": [ // Dimension
> 				1024, // Width
> 				1024, // Height
> 			],
> 				// (With this example the number is by the Nth power of two, which mean it can change the method of encoding to reduce in it's size).
> 				// However, if the dimension is not by the Nth power of two, the compression method can not be done
> 			"format": 0, // texture format, 0 i.e. rgba_8888_o
> 			"row_byte_count": 4096, // number of bytes per pixel row at texture level
> 		},
> 	},
> },
> ```

The decoded PNG image is usually an atlas, which is a large image made by merging many small images (also called sprites, Sprite).
When modifying and creating images, it is often necessary to split an atlas into sprites, or merge sprites into an atlas.

```json
{
  "size": [1024, 1024],
  "sprite": {
    "image_1": {
      "position": [0, 0],
      "size": [100, 200]
    },
    "image_2": {
      "position": [100, 200],
      "size": [700, 500]
    }
  }
}
```

When you need to split an atlas, first create an atlas manifest file `<name>.atlas.json` with the same name for the atlas file you need to split, and forward the manifest file to the tool, select the `Image atlas unpacking` function, and the tool will export the splitd sprite files to the `<name>.sprite` directory with the same name as the atlas file. >.sprite` directory.

When you need to merge the image set, you also need to create an image set list file `<name>.atlas.json` with the same name for the image set directory `<name>.sprite`, forward the list file to the tool, select the `Image image set packing` function, the tool will merge the image set files and output them to the `<name>. atlas.png` file.

The tool also supports automatic packing of sprites without the need to manually compile a manifest file. The user should forward the sprite directory `<name>.sprite` that needs to be merged to the tool, which will analyze all PNG files in the directory, automatically generate the set list files, and merge them into a set file.

#### PAM

`PAM = Popcap AniMation` is an animation format used by some PopCap games, including PvZ-2.

`TwinStar Toolkit` is a tool strictly asking for version number, so the version number of PvZ-2 PAMs are 6 for both PvZ-2 International version & PvZ-2 Chinese version for China mainland.

PAM animations are frame-by-frame, the game does not automatically fill frames, and PAM does not contain the image data used for the animation, but references the sprite resources in the PTX atlas file recorded in `properties/resources.rton` by their resource ID.

The PAM not only describes the animation itself, but can also control the behavior of the host object through commands . For example, the pea shooter's firing action is triggered by the `use_action` command used in its animation file `peashooter.pam`, and the user can try to add commands to trigger array object actions and play audio.

PAM is in binary format, which is not good for human to read and modify, you can use the tool to decode PAM to JSON to read or modify, and then re-encode the modified JSON to PAM.

The tool also supports converting PAM to Adobe Flash animation format (XFL), the user needs to use the tool to decode PAM to PAM.JSON first, then convert PAM.JSON to PAM.

XFL converted from PAM.JSON does not contain the required image data, so the user needs to extract the required sprites from the RSB and place them in the `LIBRARY/media` directory under the XFL directory; at the same time, you also need to forward the XFL directory to the tool and select the `PopCap Animation Flash Image Flash Resize` function to reset the image resolution preset in XFL according to the resolution specification of the extracted sprites (1536, 768, 384).

In addition, you can also use the Helper of the tool to easily navigate through the PAM animations, as described in the [Documentation](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document/blob/master/chinese/usage.md#%E4%BD%BF%E7%94%A8-Helper) 。It is recommend to use [TwinStar - Helper](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document/releases/tag/Helper) to view animation rather than SPC-Util Web because it's processor is very fast.

When installing [TwinStar - Helper](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document/releases/tag/Helper) msix packages, the Windows will falsely said the author of the application is untrusted. You need to go to `Properties` and add the author `TwinStar` to trusted people on your local machine to install & update TwinStar - Helper.

#### POPFX

`POPFX = POPcap eFfect` is a rendering configuration format used by some PopCap games, including PvZ-2.

In PvZ-2, POPFX is located in the RenderEffects resource group in the main data package.

By modifying POPFX, you can achieve the effect of changing the color of the whole game screen; you can decode and encode POPFX files using MOD tools.

Modification of these files is not very meaningful, so no further explanation will be given.

#### WEM

`WEM = Wwise Encoded Media` is the media (audio) format used by the Wwise audio engine, on which the audio part of PvZ-2 is based.

WEM is a media file encoded and encapsulated by Wwise and cannot be played directly, but can be decoded it by ToolKit (also need the `ffmpeg` and `ww2ogg`) to get a playable WAV file.

If you need to add custom audio to the game, you will need to encode the WAV audio into WEM format, which requires the use of Wwise software.

#### BNK

`BNK = wwise sound BaNK` is the binary encoding of the Wwise project in the Wwise audio engine, on which the audio part of PvZ-2 is based.

BNK is a binary format with a lot of data that is difficult to read and modify, and also contains some embedded WEM files. BNK can be decoded to BNK.BUNDLE using a tool, and then re-encoded to BNK after modifying BNK.BUNDLE.

There are three versions of Wwise used in PvZ-2: early version number 88, mid version number 112, and version number 140 from version 10.0.

The relationship between the Wwise software version and the version number values in the BNK is shown in the following table:

| release | version |
| :-----: | :-----: |
|    ?    |   88    |
| 2014.1  |   112   |
| 2015.1  |   113   |
| 2016.1  |   118   |
| 2016.2  |   120   |
| 2017.1  |   125   |
| 2017.2  |   128   |
| 2018.1  |   132   |
| 2019.1  |   134   |
| 2019.2  |   135   |
| 2021.1  |   140   |
| 2022.1  |   145   |

So far, the tool is able to fully parse the 140 version of BNK, but for the 88 and 112 versions, the tool is unable to decode the HIRC, ENVS and other data segments, but simply lists the binary data of each HIRC item.

The BNK is essentially a binary encoding of the Wwise project, and the process of transferring the Wwise project to the BNK is equivalent to compiling the source code in software development, while decoding the BNK is equivalent to reversing the Wwise project, so that the player can understand the structure of the PvZ-2 Wwise project through the information obtained from the decoding, and then realize the audio modification of PvZ-2.

At the same time, players can also try to recover the PvZ-2 Wwise project by reverse Wwise development through decoding BNK, but the volume of the PvZ-2 Wwise project is not small, and it need many time to recover the Wwise project based on the reverse analysis of BNK.

If you want to modify the BNK, it is recommended to download the Wwise software to understand the actual structure of the Wwise project.

Here are some things to know about Wwise BNK:

- BNK is the binary code of a Wwise project. Inverting BNK gives you a lot of information about the Wwise project, in which the game music is a combination of many sound objects, sound containers, audio buses, events and other data.

- Some objects have IDs defined by the user (called named IDs), such as event IDs (commonly play_xxx, stop_xxx), and some objects have user-defined names, but the actual IDs are automatically generated by the system as meaningless GUIDs (GUIDs are transparent to the user). ShortID is the fnv hash value of ID, ShortID with name ID is the hash value of object name, although the hash value is not reversible, but if you know the name of the object in advance, you can calculate the hash of the name to correspond with the ShortID in BNK, to achieve the effect of pseudo-inverse deduction The numeric code of WEM audio is the ShortID of the GUID automatically assigned by the system after the audio is imported into wwise, so we can't deduce the original name of the wem audio at all.

- Wwise objects are mainly of the following types:

  - `AudioDevice`

  - `AudioBus` 、`AuxiliaryAudioBus`

  - `Sound` 、`SoundPlaylistContainer` 、`SoundSwitchContajner` 、`SoundBlendContainer`

  - `MusicTrack` 、`MusicSegment` 、`MusicPlaglistContainer` 、`MusicSwitchContainer`

  - `LFO|Envolope|Time Modulator`

  - `Event` 、`EventAction`

  - `Attenuation`

  - `GameParameter` 、`Switch` 、`State`

  Where `GameParameter` 、`Switch` 、`State` 、`EventAction` 、`Bus` 、`AudioDevice` 、`Modulator` 、`Attenuation` and the other objects have unnamed IDs.

#### TXT

`TXT = TeXT`

Some XML files are present in RSB packets.

In earlier versions of PvZ-2, TXT files were used to store localized text data (`LawnStrings.txt`) (which is not using "utf-8" but "utf16-le"), but the current version has stored localized text as RTON in the Packages resource group of RSB.

There is little point in modifying such files other than localized text, so no further explanation is provided.

#### XML

`XML = eXtensible Markup Language` is a widely used data interchange format.

Some XML files are present in RSB packets.

Modifications to this type of file are not significant and are not described in more detail.

#### JSON

`JSON = JavaScript Object Notation` is a widely used data interchange format.

Some of the RtObject object data in the CDN distribution directory is not directly encoded as RTON, but is stored as textual JSON; however, some of these files have a file extension of `.json`, but are essentially RTON files.
