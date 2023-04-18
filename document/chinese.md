# PvZ-2 MOD 技术简述与入门引导

此文档是对 PvZ-2 MOD 制作技术的简单概述，为希望入门 MOD 制作的玩家提供一个粗略的引导。

- [PvZ-2 MOD 技术简述与入门引导](#pvz-2-mod-技术简述与入门引导)
	- [游戏版本](#游戏版本)
	- [应用组成与修改路线](#应用组成与修改路线)
	- [程序修改](#程序修改)
		- [概述](#概述)
		- [实例](#实例)
	- [资源修改](#资源修改)
		- [资源文件的位置](#资源文件的位置)
		- [资源文件的类别](#资源文件的类别)
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

## 游戏版本

根据运行平台的不同，游戏分为 `Android` 版与 `iPhone` 版。

根据面向区域的不同，游戏分为以下三版本：

* `ROW = Rest Of World` ：国际版，面向绝大多数地区，由西雅图宝开负责开发。 

* `NA = North America` ：北美版，面向北美，与 `ROW` 在游戏体验上几乎没有差异，仅仅是面向的地区不同。 

* `CN = ChiNa` ：中国版，面向中国大陆/内地，由上海宝开（曾经的运营方，现已解散）与拓维（目前的运营方）负责开发，与国际版在游戏体验上差异极大。

由于 `NA` 与 `ROW` 几乎没有差异，因此有时二者被统一称为国际版，本文档中也会以 `ROW` 同时代称 `ROW` 与 `NA` 两个版本。

本文档主要基于 `Android - ROW` 版，其他版本则大同小异，不做额外说明。

## 应用组成与修改路线

对于 `Android - ROW` 版游戏，游戏应用由以下部分组成：

* `<APK>` 应用安装包，实质是特殊处理的 `ZIP` 文件。
	
	* `lib` 本机库目录，其中包含有 `SO = Shared Object` 文件，这是 Linux 下的动态库文件，包含了 CPU 能直接执行的机器指令。

		* `armeabi-v7a` 面向 32 位 arm 设备的本机库目录。

			* `libPVZ2.so` 游戏的主要业务代码的编译产物。

			* `...` 其他的动态库文件。

		* `arm64-v8a` 面向 64 位 arm 设备的本机库目录，内容与 `armeabi-v7a` 一致。

	* `classes<n>.dex` Dalvik 虚拟机的可执行文件，在游戏启动时会调用 `libPVZ2.so` 负责游戏的运行。

	* `...` 其他杂项文件。

* `<Internal-Storage>` 设备的内部存储空间，实际路径为 `/data/user/<id>` 。

	* `<package-name>` 应用的内部存储空间，对于 `ROW` 版而言不必关注。

* `<External-Storage>` 设备的外部存储空间，实际路径为 `/storage/emulated/<id>` 。

	* `Android`

		* `data/<package-name>` 应用的外部存储空间。

			* `files/No_Backup` 游戏的存档目录。

				* `locao_profile` 本地配置，保存了一些游戏设置，本质是 RTON 文件。

				* `pp.dat` 用户存档，本质是 RTON 文件，`pp = Player Profile` 。

				* `snapshot<id>.dat` 用户存档的快照，本质是 RTON 文件，每次修改游戏存档后都应删除。

				* `CDN.<major>.<minor>` 对于当前游戏版本的 CDN 分发，其中的文件将在运行时覆盖数据包中 `Packages` 资源群内的对应文件。

				* `...` 其他杂项文件。

		* `obb/<package-name>` 应用的扩展数据包存储目录。

			* `main.<version-code>.<package-name>.obb` 游戏的扩展数据包，包含了图像、动画、文本等游戏使用的所有资源，本质是 RSB 文件。

根据修改目标的不同，游戏的修改路线分为以下两类：

* 程序修改：对游戏的程序本体（即 APK ，特别是其中的 DEX 与 SO）进行修改，对修改者技术要求较高，需要具备一定的软件逆向知识，可以自定义游戏的运行逻辑，理论上可以做出任何效果。

* 资源修改：对游戏的资源数据（主要是扩展数据包与用户存档）进行修改，对修改者技术要求低，可以通过已有的工具程序对这些资源文件进行编、解码，只能在程序本体制定的规则下进行修改，具有一定的局限性。

## 程序修改

### 概述

程序修改即是对游戏的程序本体进行修改。程序修改可以自定义游戏的运行逻辑，理论上可以做出任何效果。

程序修改的本质是对程序的逆向工程，原理如下：

1. 软件开发阶段的源代码在软件分发时会进行编译，以生成 CPU 或解释器能直接执行的代码。

	* 对于 `C` 、`C++` 等 Native 语言，源代码会编译为面向特定平台 CPU 的机器码。

	* 对于 `Java` 等语言，源代码会编译为面向特定解释器（`JVM` 等）的字节码。

2. 源代码的编译产物决定了程序的执行逻辑，修改这些编译产物，就可以改变程序的执行逻辑，让程序做出修改者所希望的行为。

3. 对这些编译产物的分析与修改，称之为软件逆向，需要修改者具备 x86/arm 汇编、smail 等知识。

> 这里需要纠正一个常见的谬误：程序修改是修改游戏的源代码。
> 
> 一些人不具备软件开发的相关知识，错误地将对源代码编译产物的修改视为对源代码本身的修改；\
> 源代码处于开发层面而非用户层面，只有游戏运营方能获取并修改源代码，第三方只能通过黑客手段盗取源代码，或根据逆向分析模仿/复原出源代码（对技术要求高，并且工作量很大），目前为止没有第三方能做到二者之一，故而没有第三方能对游戏源代码进行修改，只能对源代码的编译产物进行修改。

程序修改的主要对象是 APK 中的 DEX 文件与 SO 文件，其中，`libPVZ2.so` 包含了游戏的主要业务代码。可以借助 IDA 、dex2jar 等工具对这些文件进行逆向分析与修改。

逆向工程本身是一个复杂的领域，本文档不做更多说明，有兴趣者可以在网络上查找资料自行学习。

### 实例

* 零阳光无冷却

	通过修改 `libPVZ2.so` ，可以将植物种植所需的阳光数目与冷却时间设为零。
	
	一种传统而简单的修改方式是删除 so 中的 `Cost` 与 `PacketCooldown` 字符串常量，使游戏在反序列化 RTON 数据为 `RtObject` 对象时无法获取到阳光数目与冷却时间的数据，只能使用默认值 `0` ，从而呈现出零阳光无冷却的效果。

* 修改游戏帧率

	PvZ-2 的屏幕刷新率由 DEX 控制，在 DEX 中，程序将每间隔 `33 ms` 对屏幕进行一次刷新，由此得到了 `30 fps` 的帧率，可以修改刷新间隔修改为 `17 ms` 以达到 `60 fps` 的帧。
	
	并不推荐这么做：一来，高频率刷新对设备会有一定的负担；二来，游戏中的许多动画是编码为 `30 fps` 或 `24 fps` 的，修改刷新率并不能改变这些动画的实际帧率，必须对动画文件进行补帧才能实现高帧率动画，而这需要很大的工作量，并不现实。

* 修改字符缓冲区大小

	PvZ-2 国际版在文本渲染时存在有限的字符缓冲区。由于国际版本身只面向英语、法语、西语等六种语言的地区，这六类语言的字符种类少，预设的字符缓冲区足够使用；但对于中文这类字符种类极多的语言而言，预设的字符缓冲区不足以容纳丰富的字符种类，这导致了 PvZ-2 国际版的汉化 MOD 在游戏时容易出现字符消失乃至程序崩溃的问题。

	可以尝试修改游戏的字符缓存区容量，以缓解该问题，相关的程序代码位于 `libPVZ2.so` 中 `Sexy::PrimeGlyphCache` 类的一个虚函数中。可以参照我在 BiliBili 上的 [个人专栏](https://www.bilibili.com/read/cv16512322?spm_id_from=333.999.0.0) ，但其中介绍的方法只对低端设备（768、384）设备有效，对高端设备（1536）无效。

* 解除旧版本中的未完成世界限制

	在 PvZ-2 国际版的早期版本（1.4 左右），游戏实现了部分 Far-Future 与 Dark-Ages 世界的编码与资源，但在交付给用户的程序本体中禁用了这两个世界，玩家无法通过仅修改 RSB 数据包来调取出这两个世界。

	通过修改 `libPVZ2.so` ，可以解除这一限制。

* 私服创建

	通过对程序的抓包、逆向，可以分析出游戏与服务器通信的规则，从而构建出自己的私服。

* `Project Paradox`

	`Paradox` 是一个 PvZ-2 MOD 项目，其利用了程序修改的手段做出了一些很有意思的效果，以下是对该项目修改原理的简单说明：

	1. 在安装包中引入自编译的 `LawnMod.so` 动态库文件。

	2. 修改 `DEX` 文件，让游戏加载 `LawnMod.so` ，并调用其中的初始化函数。

	3. 在 `LawnMod.so` 的初始化函数中，对已加载到内存中的 `libPVZ2.so` 中的诸多代码段进行 HOOK ，实现对游戏运行逻辑的自定义。

		例如，该项目对 `libPVZ2.so` 中关于植物数字编码的代码段进行 HOOK ，使用自定义的函数替代原函数执行，并在自定义函数中首先调用原函数，然后定义额外的植物数字编码，由此在游戏中添加了自定义的植物数字编码，使得玩家可以获取并在存档中保有该 MOD 的自定义植物。

	该 MOD 的修改原理并不特别，但难点在于如何撰写 HOOK 逻辑，这需要修改者对 `libPVZ2.so` 进行足够的逆向分析。

## 资源修改

资源修改即是对游戏的资源数据进行修改，资源数据指游戏除程序本体外的所有数据文件，对于 PvZ-2 而言，资源数据主要指游戏的扩展数据包与存档目录下的各文件。

资源修改只能在程序本体制定的规则下进行修改，具有一定的局限性。

得益于 PvZ-2 MOD 辅助工具的发展，资源修改的门槛很低，修改者只需要了解基本的计算机知识即可着手修改。资源修改所需的辅助工具可以从 [Fandom wiki](https://ernestoam.fandom.com/wiki/Plants_vs._Zombies_2_Hacking_Tools) 上获取，本文档使用的辅助工具为 [`TwinStar ToolKit`](https://github.com/twinkles-twinstar/TwinStar.ToolKit) ，请读者先按照该工具的 [文档](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document) 进行安装。

下文中的“工具”均指代 `TwinStar ToolKit` ，不过，你也可以尝试使用其他的开放工具进行文件处理。

### 资源文件的位置

根据资源文件在存储空间中的位置，分为以下两类：

* 包内资源：游戏的绝大部分资源都被打包在单一的 RSB 数据包文件中，该 RSB 文件以应用扩展数据包的形式存储于应用的 OBB 目录内。对包内资源的修改需要对 RSB 进行解包与重打包。

* 包外资源：游戏的少部分资源松散地存储在游戏的存档目录中，包括用户存档 `pp.dat` 、本地配置 `local_profile` 、未完成关卡缓存 `saves` 、CDN 分发 `CDN.<major>.<minor>` 、等。

### 资源文件的类别

PvZ-2 中使用的资源文件种类较多，且多是 PopCap 自研的私有格式，需要借助辅助工具对这些文件进行解码与再编码。

包内资源的文件扩展名与其类型保持一致，但包外资源的文件扩展名并不如此，比如 `pp.dat` 与 `local_profile` 文件都是 RTON 文件却不使用 `.rton` 扩展名、而 CDN 分发目录中的部分 RTON 文件又以 `.json` 作为扩展名。在

#### RSB & RSG

`RSB = Resource Stream Bundle` 与 `RSG = Resource Stream Group` 是 PvZ-2 在内的一些 PopCap 使用的资源数据包格式。

RSG 是对资源的封装，但 RSG 本身无法完整地描述出资源信息，必须内嵌于 RSB 中；RSB 是对 RSG 的封装，其中描述了完整的资源信息，并根据分辨率、区域差异对资源文件进行分组，使游戏能够根据设备实际情况选择性地加载所需资源。

在 PvZ-2 中，只存在唯一的 RSB 文件，即应用的扩展数据包文件 `<External-Storage>/Android/obb/<package-name>/main.<version-code>.<package-name>.obb` 。

在 PvZ-2 中，RSG 不会以单文件的形式存在，而是内嵌于 RSB 数据包中。

RSB 保存了游戏所使用的各类资源文件，可以使用工具对 RSB 进行解包与打包，修改者可以在程序本地允许的机制下删除其中的资源，或添加自定义资源。

#### RTON

`RTON = ReflecTion Object Notation` 是 PvZ-2 在内的一些 PopCap 使用的对象标记格式。

RTON 是 PopCap SexyFramework （PvZ-2 所基于的游戏开发框架）中 `Sexy::RtObject` 对象的二进制表示；从 RtObject 对象编码为 RTON 的过程称为序列化，从 RTON 解码为 RtObject 对象的过程则称为反序列化。

RTON 是二进制格式，不利于人类阅读与修改，因此一般不直接对 RTON 进行修改，而是借助 MOD 工具将 RTON 解码为 JSON ，对文本化的 JSON 文件进行修改，再使用 MOD 工具将 JSON 重编码为 RTON 。

在 PvZ-2 中文版中，RTON 文件经过了一层 Rijndael 加密。

工具支持对 RTON 的编码、解码、加密、解密，其中加密与解码所需的密钥需要用户自行从游戏程序文件中寻找。

RTON 的基本组成如下（解码为 JSON 后）：

```json
{
	"version": 1, // RTON 版本，始终为 1
	"objects": [ // 该 RTON 文件所记录的 RtObject 对象
		{ // 一个 RtObject 对象
			"uid": "1.2.33445566", // 该对象的 Unique-ID ，可以通过 RTID(uid@rton_file_name) 引用该对象 
			"aliases": [ "XX" ], // 该对象的别名，可以通过 RTID(alias@rton_file_name) 引用该对象
			"objclass": "...", // 该对象的实际类型
			"objdata": { // 该对象的属性，根据 objclass 的不同，objdata 内的属性也有所不同
				"property_1": true, // 一个 boolean 类型的属性
				"property_2": 2.4, // 一个 number 类型的属性
				"property_3": "a string value", // 一个 string 类型的属性
			},
		},
		// ...
	],
}
```

可以通过 [PvZ2LibraryAnalyzer 项目](https://github.com/twinkles-twinstar/TwinStar.ToolKit.PvZ2LibraryAnalyzer) 对游戏的 `libPVZ2.so` 进行分析，解析出 RTON 的类属性（objdata）编写规范，这对 RTON 修改会很有帮助。

你也可以在工具的 [Miscellaneous 分发](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document/releases/tag/Miscellaneous) 中找到 9.9.1 版本的 RTON 类属性规范表。

#### PTX

`PTX = Popcap TeXture` 是 PvZ-2 在内的一些 PopCap 使用的图像纹理格式。

工具可以将 PTX 纹理解码为 PNG 图像，或将 PNG 图像编码为 PTX 纹理。

PTX 中一般只存储图像的纹理数据，不包含图像尺寸与纹理格式信息，这些信息被记录在 PTX 所在的 RSB 数据包中，使用工具解包 RSB 数据包后，可以在 `*.rsb.bundle` 目录内的 `manifest.json` 工程清单文件中查看各 PTX 文件对应的图像尺寸与纹理格式。

RSB 工程清单文件中记录的 `format` 字段是 PTX 中存储的纹理格式类型的序号值，其与纹理格式类型的对应关系见下表：

| 版本    | 序号 | 纹理格式           |
|:-------:|:----:|:------------------:|
| iPhone  |    0 | argb_8888          |
| Android |    0 | rgba_8888_o        |
| any     |    1 | rgba_4444          |
| any     |    2 | rgb_565            |
| any     |    3 | rgba_5551          |
| any     |   21 | rgba_4444_tiled    |
| any     |   22 | rgb_565_tiled      |
| any     |   23 | rgba_5551_tiled    |
| any     |   30 | rgba_pvrtc4        |
| any     |   31 | rgba_pvrtc2        |
| any     |   32 | rgb_etc1           |
| any     |   33 | rgb_etc2           |
| any     |   34 | rgba_etc2          |
| any     |   35 | rgb_dxt1           |
| any     |   36 | rgba_dxt3          |
| any     |   37 | rgba_dxt5          |
| any     |   38 | rgb_atitc          |
| any     |   39 | rgba_atitc         |
| ROW     |  147 | rgb_etc1_a_8       |
| CN      |  147 | rgb_etc1_a_palette |
| any     |  148 | rgb_pvrtc4_a_8     |
| iPhone  |  149 | argb_8888_a_8      |
| Android |  149 | rgba_8888_o_a_8    |
| CN      |  150 | rgb_etc1_a_palette |

> 部分纹理格式并未实际应用在 PvZ-2 游戏中，而是通过对 `libPVZ2.so` 的逆向分析得以发现。
> 
> 部分纹理格式带有 `_o` 后缀，代表该纹理数据是端序无关的（数据逐字节编码），在 OpenGL 、libpng 中，RGBA_8888 纹理是端序无关的，数据按照 R 、G 、B 、A 的顺序逐字节排列。

使用工具将 PTX 解码为 PNG 时，需要提供图像的宽度、高度、纹理格式，这些信息都可以在 RSB 工程清单文件中找到；而在将 PNG 编码为 PTX 时，只需要提供纹理格式。

一些纹理格式对图像的尺寸有所要求（如 ETC 要求图像为正方形、宽高均为二的N次幂），工具在编码与解码 PTX 时会工具用户提供的尺寸值自动填充到所需尺寸。

> 例如，以下是 Android RSB 工程清单文件中描述的一个 PTX 文件信息：
> 
> ```json
> "ATLASES/ALWAYSLOADED_1536_00.PTX": { // PTX 文件在 RSB 中的包内路径
> 	"additional": {
> 		"type": "texture",
> 		"value": {
> 			"size": [
> 				1024, // 图像宽度
> 				1024, // 图像高度
> 			],
> 			"format": 0, // 纹理格式，0 即 rgba_8888_o
> 			"row_byte_count": 4096, // 纹理层面上每行像素的字节数
> 		},
> 	},
> },
> ```

解码得到的 PNG 图像一般是图集（Atlas），图集是许多小图像（也称精灵，Sprite）合并而成的大图像，在进行图像修改与制作时，往往需要将图集分解为精灵，或将精灵合并为图集。

工具提供对图集的分解与合并功能，使用该功能时，需要准备一份图集清单文件 `*.atlas.json` ，在其中描述出图集中所有精灵的定位与尺寸信息，例如以下示例：

```json
{
	"size": [ 1024, 1024 ],
	"sprite": {
		"image_1": {
			"position": [ 0, 0 ],
			"size": [ 100, 200 ],
		},
		"image_2": {
			"position": [ 100, 200 ],
			"size": [ 700, 500 ],
		},
	},
}
```

需要分解图集时，首先应为需要分解的图集文件 `<name>.atlas.png` 创建同名的图集清单文件 `<name>.atlas.json` ，并将该清单文件转发给工具，选择 `Image 图集解包` 功能，工具会将分解得到的精灵图文件输出至与图集文件同名的 `<name>.sprite` 精灵图目录内。

需要合并图集时，同样要为需要合并的精灵图目录 `<name>.sprite` 创建同名的图集清单文件 `<name>.atlas.json` ，并将该清单文件转发给工具，选择 `Image 图集打包` 功能，工具会将合并出图集文件，并输出为与精灵图目录同名的 `<name>.atlas.png` 文件。

工具也支持对精灵图的自动打包，而无需手工编写图集清单文件。用户应将需要合并的精灵图目录 `<name>.sprite` 转发给工具，工具将分析该目录下所有 PNG 文件，自动生成的图集清单文件，并将它们合并为图集文件。

#### PAM

`PAM = Popcap AniMation` 是 PvZ-2 在内的一些 PopCap 游戏所使用的一种动画格式。

PAM 动画是逐帧的，游戏不会自动补帧，PAM 中不包含动画所使用的图像数据，而是通过资源 ID 引用 `properties/resources.rton` 中记录的 PTX 图集文件中的精灵图资源。

PAM 不仅描述了动画本身，也能通过指令控制宿主对象的行为 。例如，豌豆射手的发射动作是由其动画文件 `peashooter.pam` 中使用的 `use_action` 指令触发的，用户可以尝试添加指令来触发数组对象动作、播放音频。

PAM 是二进制格式，不利于人类阅读与修改，可以使用工具将 PAM 解码为 JSON 以阅读或修改，再将修改后的 JSON 重新编码为 PAM 。

工具还支持将 PAM 转换为 Adobe Flash 动画格式（XFL），用户需要先使用工具将 PAM 解码为 PAM.JSON ，再将 PAM.JSON 转换为 PAM.XFL ，可以在 PAM.XFL 中进行修改，再使用工具将 PAM.XFL 重新转换为 PAM.JSON 与 PAM 。

由于 PAM 本身不包含图像信息，因此 PAM.JSON 转换所得的 PAM.XFL 不包含所需的图像数据，用户还需要从 RSB 中提取所需的精灵图，并将精灵图放置于 XFL 目录下的 `LIBRARY/media` 目录内；同时，也需要将 XFL 目录转发给工具，选择 `PopCap Animation Flash 图像分辨率调整` 功能，根据提取的精灵图的分辨率规格（1536、768、384）重设 XFL 中预设的图像分辨率。

此外，也可以使用工具的 Helper 模块方便地浏览 PAM 动画，具体参阅该 [文档](https://github.com/twinkles-twinstar/TwinStar.ToolKit.Document/blob/master/chinese/usage.md#%E4%BD%BF%E7%94%A8-Helper) 。

#### POPFX

`POPFX = POPcap eFfect` 是 PvZ-2 在内的一些 PopCap 游戏所使用的一种渲染配置格式。

在 PvZ-2 中，POPFX 位于主数据包中 RenderEffects 资源群中。

通过修改 POPFX ，可以实现游戏画面整体变色的特效；可以使用 MOD 工具对 POPFX 文件进行解码与编码。

对该类文件的修改意义不大，故不作更多说明。

#### WEM

`WEM = Wwise Encoded Media` 是 Wwise 音频引擎所使用的媒体（音频）格式，PvZ-2 的音频部分基于该引擎。

WEM 是经过 Wwise 编码与封装的媒体文件，无法直接播放，可以使用工具对 WEM 文件进行解码，得到可播放的 WAV 文件，工具的解码需要借助另外的开源项目 `ffmpeg` 与 `ww2ogg` 。

如果需要在游戏中加入自定义的音频，就需要将 WAV 音频编码为 WEM 格式，这个过程需要使用 Wwise 软件，具体教程可以在网络上找到，本文件不做额外说明。

#### BNK

`BNK = wwise sound BaNK` 是 Wwise 音频引擎中对 Wwise 工程的二进制编码，PvZ-2 的音频部分基于该引擎。

BNK 是二进制格式，其中的数据十分繁杂，难以阅读与修改，同时也包含了一些内嵌的 WEM 文件。可以使用工具将 BNK 解码为 BNK.BUNDLE ，对 BNK.BUNDLE 进行修改后再重新编码为 BNK 。

BNK 有版本之分，不同版本的 Wwise 软件会生成不同版本的 BNK 文件。PvZ-2 使用的 Wwise 有四个版本：早期版本编号为 72 与 88 ，中期版本编号为 112 ，10.0 版本开始编号为 140 。

Wwise 软件版本与 BNK 中的版本序号值对应关系见下表：

| 版本   | 序号 |
|:------:|:----:|
|    ... |  ... |
| 2010.1 |    ? |
| 2010.2 |    ? |
| 2010.3 |   53 |
| 2011.1 |    ? |
| 2011.2 |    ? |
| 2011.3 |    ? |
| 2012.1 |    ? |
| 2012.2 |   72 |
| 2013.1 |    ? |
| 2013.2 |   88 |
| 2014.1 |  112 |
| 2015.1 |  113 |
| 2016.1 |  118 |
| 2016.2 |  120 |
| 2017.1 |  125 |
| 2017.2 |  128 |
| 2018.1 |  132 |
| 2019.1 |  134 |
| 2019.2 |  135 |
| 2021.1 |  140 |
| 2022.1 |  145 |

工具支持对 72 与 88 至 145 版本的 BNK 进行编码与解码，并能够完全解析 HIRC 数据段，但不保证解析是完全正确的。

BNK 的本质是 Wwise 工程的二进制编码，Wwise 工程到 BNK 的过程相当于软件开发中的源代码编译，而解码 BNK 就相当于对 Wwise 工程的逆向，玩家可以通过解码得到的信息了解 PvZ-2 Wwise 工程的结构，进而实现 PvZ-2 的音频修改。

同时，玩家也可以通过对 BNK 的解码进行逆向 Wwise 开发，尝试复原出 PvZ-2 的 Wwise 工程，不过 PvZ-2 的 Wwise 工程体量不小，想要根据对 BNK 逆向分析来复原出 Wwise 工程，需要的工作量不是一两个人花三四天就能完成的，需要具备一定的耐心与恒心。

如果你想要修改 BNK ，那么推荐下载 Wwise 软件以了解 Wwise 工程的实际结构。

下面是关于 Wwise BNK 的一些知识：

* BNK 是 Wwise 工程的二进制编码，对 BNK 逆向就可以得到 Wwise 工程的许多信息，在 Wwise 工程中，游戏音乐是由许多声音对象、声音容器、音频总线、事件等数据组合而成的。

* Wwise 内的对象都具有 ID，有些对象的 ID 由用户自己定义（称为具名 ID），例如事件 ID（常见的有 play_xxx，stop_xxx），另一些对象虽然也能被用户定义名字，但实际 ID 是由系统自动生成无意义的 GUID（GUID 对用户是透明的）。以上两类 ID，在 BNK 中都会会被编码为 ShortID（32 位数），ShortID 是 ID 的 fnv 散列值，具名 ID 的 ShortID 就是对象名称的散列值，虽然散列值不可逆，但如果预先知道对象的名称，就能计算名称的散列来和 BNK 中的 ShortID 对应，实现伪反推的效果；而不具名 ID 的 ShortID 是随机的 GUID 的散列值，我们无法从这类 ShortID 推导出对象原本的名称。WEM 音频的数字编码就是音频导入 wwise 后，系统自动分配的 GUID 的 ShortID，因此我们完全无法反推出 wem 音频原有的名字。

* Wwise 对象主要有以下类型：

	* `AudioDevice`

	* `AudioBus` 、`AuxiliaryAudioBus`

	* `Sound` 、`SoundPlaylistContainer` 、`SoundSwitchContajner` 、`SoundBlendContainer`

	* `MusicTrack` 、`MusicSegment` 、`MusicPlaglistContainer` 、`MusicSwitchContainer`

	* `LFO|Envolope|Time Modulator`

	* `Event` 、`EventAction`

	* `Attenuation`

	* `GameParameter` 、`Switch` 、`State`

	其中，`GameParameter` 、`Switch` 、`State` 、`EventAction` 、`Bus` 、`AudioDevice` 、`Modulator` 、`Attenuation` 拥有具名 ID，其他对象则是不具名 ID。

#### TXT

`TXT = TeXT`

RSB 数据包中存在部分 XML 文件。

在早期版本的 PvZ-2 中，使用 TXT 文件存储本地化的文本数据（`LawnStrings.txt`），但目前版本已经将本地化文本以 RTON 的形式存储与 RSB 的 Packages 资源群内。

对除本地化文本外的该类文件的修改意义不大，故不作更多说明。

#### XML

`XML = eXtensible Markup Language` 是一种广泛使用的数据交换格式。

RSB 数据包中存在部分 XML 文件。

对该类文件的修改意义不大，故不作更多说明。

#### JSON

`JSON = JavaScript Object Notation` 是一种广泛使用的数据交换格式。

CDN 分发目录下的部分 RtObject 对象数据并没有直接编码为 RTON ，而是以文本化的 JSON 存储；但其中有也些文件虽然以 `.json` 作为文件扩展名，但实质上是 RTON 文件。





