# Recipe Lab CN · Mod

> 索尼 PlayMemories 相机应用的深度定制分支：面向 **A7R2（ILCE-7RM2）** 重构按键、加入应用内回放与波轮曝光控制。
>
> A heavily customized fork of [Recipe Lab](https://github.com/voxivoid/recipe-lab-sony-pmca) for the **Sony A7R II (ILCE-7RM2)**: remapped keys, in-app playback, and dial-based exposure control.

---

## 中文说明

### 这是什么

在原版 [Recipe Lab](https://github.com/voxivoid/recipe-lab-sony-pmca)（77 套胶片配方）基础上，针对 A7R2 的实际拍摄习惯做的深度改造。原版的界面汉化来自 [jcx396905-gif/recipe-lab-sony-pmca-cn](https://github.com/jcx396905-gif/recipe-lab-sony-pmca-cn)。

**核心思路：把应用从「选配方然后退出」变成「选配方、调参数、拍照、回放」一站完成。**

### 主要改动

| 项目 | 原版 | 本分支 |
|---|---|---|
| 左右键 | 切换参数 / 芯片 | **切换品牌分组** |
| 上下键 | 切换参数行 | **切换滤镜** |
| 控制转盘（OK 外圈） | 切换参数 | **切换参数行 / 编辑时改值** |
| 前波轮 | 切滤镜 | **光圈 ±1 档** |
| 后波轮 | 选参数 | **快门 ±1 档** |
| Fn | 品牌列表 | **写入当前参数**（唯一落盘入口） |
| C1 | 开发工具菜单 | **退出应用** |
| 删除键 | 恢复出厂色彩 | **应用内回放**（回放中再按则删除） |
| 回放 | 不支持 | **支持**，含翻页与删除 |
| 曝光显示 | 无 | **常驻显示**光圈 / 快门 |

### 按键映射

| 按键 | 功能 |
|---|---|
| ← / → | 切换品牌分组 |
| ↑ / ↓ | 切换滤镜 |
| 控制转盘 | 选参数行；按 OK 进入编辑后改值 |
| OK（短按） | 选中参数 / 确认 |
| 前波轮 | 光圈 ±1 档（需 M 或 A 挡 + 电子镜头） |
| 后波轮 | 快门 ±1 档（需 M 或 S 挡 + 电子镜头） |
| S1（半按） | 对焦 |
| S2（全按） | 直接拍照 |
| Fn | **写入当前参数**（重启相机后在所有模式生效） |
| C1 | 退出应用 |
| 删除键 | 进入回放；回放中再按删除当前照片（需按两次确认） |
| 回放中 ← / → | 翻页 |
| 回放中 MENU | 退出回放，恢复取景 |

### 重要说明

**关于写入时机**：切换滤镜、调整参数都只是**预览**，退出应用会自动还原。只有按 **Fn** 才会真正写入机身存储。写入后需要**关机重开**，配方才会在 P/A/S/M 全部模式生效。

**关于写入位置**：写入的是「**创意风格**」（Creative Style）及其饱和度/对比度/锐度，加上白平衡、曝光补偿、DRO、照片效果。A7R2 没有「创意外观」（Creative Look）这个概念，那是 A7S3/A7M4 之后的机型才有的。

**关于光圈快门**：走的是 `CameraEx.incrementAperture()` / `incrementShutterSpeed()`，改的是真硬件，会进 EXIF。前提是机身处于 M 挡（或 A/S 挡对应权限）且镜头有电子触点，转接手动镜头无法控制。

**关于回放**：A7R2 的 `K_PLAY` 是固件级按键，应用拦不住，按下去会直接进入相机原生回放。所以应用内回放挂在**删除键**上。

### 安装

1. 下载 Releases 里的 `RecipeLab-mod.apk`
2. 下载 [Sony-PMCA-RE](https://github.com/ma1co/Sony-PMCA-RE/releases) 的 `pmca-console`
3. 相机设置：`MENU → 设置 → USB 连接 → 大容量存储`，插好存储卡并开机
4. 命令行执行：
   ```bash
   ./pmca-console.exe install -f RecipeLab-mod.apk
   ```
5. 出现 `Task completed successfully` 后，拔线、**关机再开机**
6. `MENU → 应用程序 → 应用程序列表 → 配方实验室`

> 首次安装时相机可能黑屏并断开连接，属正常现象。如果之后电脑完全识别不到相机，**取出电池静置 30 秒**再装回，这是唯一能彻底复位 USB 子系统的方法。

### 自行编译

需要 JDK 17 + Android build-tools 30.0.3 + platform android-28。原生库可直接从官方 APK 提取，**无需 NDK**：

```bash
unzip -o RecipeLabCN.apk "lib/armeabi/librecipelab.so" -d out/apklib
aapt package -f -m -J out/gen -M AndroidManifest.xml -S res -I android.jar
javac -encoding UTF-8 --release 8 -cp android.jar -d out/classes out/gen/**/R.java src/**/*.java
java -cp d8.jar com.android.tools.r8.D8 --release --min-api 10 --lib android.jar --output out/dex out/classes/**/*.class
aapt package -f -M AndroidManifest.xml -S res -A assets -I android.jar -F out/unaligned.apk
cd out/dex && aapt add ../unaligned.apk classes.dex && cd ../apklib && aapt add ../unaligned.apk lib/armeabi/librecipelab.so
zipalign -f 4 out/unaligned.apk out/aligned.apk
java -jar lib/apksigner.jar sign --ks debug.keystore --v1-signing-enabled true --v2-signing-enabled false --out RecipeLab-mod.apk out/aligned.apk
```

### 已知限制

- **仅适配 A7R2（固件 4.00 / 4.01）**，其他机型按键扫描码不同，需要自行调整
- 回放采用 1/8 采样解码 + 内嵌缩略图渐进加载；PMCA 应用访问存储的 I/O 带宽约 1.2MB/s，单张 4200 万像素照片完整解码需 4-5 秒，属环境限制
- 灰阶渐变区域可能存在轻微色带（`BitmapFactory` 的 IDCT 缩放精度所限）
- 光圈/快门控制依赖索尼私有 API，不同固件版本行为可能不同

### 致谢

- [voxivoid](https://github.com/voxivoid) —— 原版 Recipe Lab
- [ma1co](https://github.com/ma1co) —— PlayMemories 平台逆向（Sony-PMCA-RE、OpenMemories-Framework）
- [jcx396905-gif](https://github.com/jcx396905-gif) —— 中文汉化分支

### 许可

MIT License，继承自原项目。

---

## English

### What is this

A deeply customized fork of [Recipe Lab](https://github.com/voxivoid/recipe-lab-sony-pmca) (77 film recipes), tailored for the **Sony A7R II (ILCE-7RM2)**. The Chinese UI localization comes from [jcx396905-gif/recipe-lab-sony-pmca-cn](https://github.com/jcx396905-gif/recipe-lab-sony-pmca-cn).

**Core idea: turn the app from "pick a recipe, then quit" into a one-stop flow — pick recipe, tune parameters, shoot, and review — without leaving the app.**

### What changed

| Item | Original | This fork |
|---|---|---|
| Left / Right | cycle parameters | **switch brand group** |
| Up / Down | cycle parameter row | **switch recipe** |
| Control wheel | cycle parameters | **select parameter row / edit value** |
| Front dial | switch recipe | **aperture ±1 stop** |
| Rear dial | select parameter | **shutter ±1 stop** |
| Fn | brand browser | **write current settings** (only commit path) |
| C1 | developer menu | **quit app** |
| Delete | factory reset colors | **in-app playback** (deletes when already in playback) |
| Playback | not supported | **supported**, with paging and delete |
| Exposure readout | none | **always-on** aperture / shutter |

### Key mapping

| Key | Action |
|---|---|
| ← / → | Switch brand group |
| ↑ / ↓ | Switch recipe |
| Control wheel | Select parameter row; edit value after pressing OK |
| OK (short) | Select parameter / confirm |
| Front dial | Aperture ±1 stop (M or A mode, electronic lens required) |
| Rear dial | Shutter ±1 stop (M or S mode, electronic lens required) |
| S1 (half-press) | Autofocus |
| S2 (full-press) | Take picture |
| Fn | **Write current settings** (takes effect in all modes after restart) |
| C1 | Quit app |
| Delete | Enter playback; press again to delete current photo (double-press confirm) |
| ← / → in playback | Page through photos |
| MENU in playback | Exit playback, restore live view |

### Important notes

**When are settings written?** Switching recipes and adjusting parameters are **preview only** — quitting the app restores everything. Only **Fn** commits to the camera's persistent storage. After writing, **power-cycle the camera** for the recipe to apply in all P/A/S/M modes.

**Where are settings written?** To **Creative Style** (its saturation / contrast / sharpness), plus white balance, exposure compensation, DRO and Picture Effect. The A7R2 has no "Creative Look" — that concept arrived with the A7S III / A7 IV generation.

**Aperture & shutter** go through `CameraEx.incrementAperture()` / `incrementShutterSpeed()`. These drive real hardware and land in EXIF. Requires M mode (or A/S mode for the respective parameter) and a lens with electronic contacts; adapted manual lenses cannot be controlled.

**About playback**: `K_PLAY` on the A7R2 is a firmware-level key — the app cannot intercept it, pressing it jumps straight to the camera's native playback. In-app playback is therefore bound to the **delete key**.

### Installation

1. Download `RecipeLab-mod.apk` from Releases
2. Download `pmca-console` from [Sony-PMCA-RE](https://github.com/ma1co/Sony-PMCA-RE/releases)
3. On camera: `MENU → Setup → USB Connection → Mass Storage`, insert card and power on
4. Run:
   ```bash
   ./pmca-console.exe install -f RecipeLab-mod.apk
   ```
5. After `Task completed successfully`, unplug, **power off and on**
6. `MENU → Application → Application List → Recipe Lab`

> The camera may go black and disconnect during first install — this is normal. If the computer then cannot detect the camera at all, **remove the battery for 30 seconds**; that is the only way to fully reset the USB subsystem.

### Building from source

Requires JDK 17 + Android build-tools 30.0.3 + platform android-28. The native library can be extracted from the official APK — **no NDK needed**. See the Chinese section above for the full command sequence.

### Known limitations

- **A7R2 only (firmware 4.00 / 4.01)** — other bodies use different key scan codes
- Playback uses 1/8-sampled decoding with embedded-thumbnail progressive loading; PMCA storage I/O is roughly 1.2 MB/s, so a full 42 MP decode takes 4-5 s — an environment limit, not a code issue
- Slight banding may remain in smooth gradients (a limitation of `BitmapFactory`'s IDCT scaling)
- Aperture/shutter control relies on Sony private APIs; behavior may vary across firmware versions

### Credits

- [voxivoid](https://github.com/voxivoid) — original Recipe Lab
- [ma1co](https://github.com/ma1co) — PlayMemories reverse engineering (Sony-PMCA-RE, OpenMemories-Framework)
- [jcx396905-gif](https://github.com/jcx396905-gif) — Chinese localization branch

### License

MIT, inherited from the original project.
