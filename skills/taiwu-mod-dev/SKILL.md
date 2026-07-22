---
name: taiwu-mod-dev
description: 用于为《太吾绘卷：天幕心帷》（The Scroll of Taiwu，Steam App 838350）制作独立 C# Mod 的全流程 skill。覆盖从环境检查（.NET SDK、ilspycmd、游戏定位）、全量反编译游戏程序集、版本一致性校验、编写插件入口、用 HarmonyLib 打补丁、定义 Config.Lua 设置项、前后端 RPC 通信，到编译部署、看日志调试、发布到创意工坊及后续版本更新维护的完整链路。当用户提到太吾绘卷、The Scroll of Taiwu、taiwu mod、为太吾写 mod、改太吾游戏行为、Harmony patch 太吾、TaiwuRemakePlugin、反编译太吾、更新/发布太吾 mod、太吾游戏更新后 patch 失效/适配、太吾 mod 的 .NET 版本/TargetFramework 等任何相关场景时使用——即使用户没说"用这个 skill"。本 skill 不依赖任何特定工作目录。
---

# 太吾绘卷独立 Mod 开发

辅助用户为《太吾绘卷：天幕心帷》制作**独立、自包含**的 C# Mod。任何工作目录下都能用。

## 何时用这个 skill

凡涉及"给太吾写 mod / 改游戏行为 / 加功能 / 调数值 / 修 bug / 反编译太吾查代码 / 发布更新 mod / 适配游戏新版本"都用。包括："让太吾免疫中毒""加个物品""改战斗伤害""mod 加载不了""Harmony patch 不生效""帮我反编译看看某方法签名""怎么更新已发布的 mod""游戏更新了我的 mod 还能用吗"等。

## 总流程：四阶段

任何"做太吾 mod"的任务都按这个顺序推进。**开发前先确保前三步就绪**（环境、反编译源码、引用 dll），它们是开发的前提；开发完成后走阶段四发布与持续维护。

1. **前置检查** → 确保 .NET 8+ 开发环境就绪、ilspycmd 版本匹配、定位到游戏安装目录。见下方"阶段一"。
2. **反编译就绪** → 确保有与当前游戏版本一致的反编译源码供阅读。见下方"阶段二"。
3. **开发** → 写插件入口 / Harmony patch / Config.Lua，编译部署，看日志。见下方"阶段三"和各 reference。
4. **发布与维护** → 完善 Config.Lua（与用户交互）、自检、游戏内上传创意工坊；以及发布后的版本更新、适配游戏新版本。见下方"阶段四"和 `references/publishing.md`。

> **理解游戏机制时，配合「游戏机制知识库」（阶段二可选，推荐）**：把游戏内《太吾百晓册》（官方百科）转成 markdown，让 AI 先理解机制/数值怎么设计，再结合反编译源码定位实现——patch 写得更准、少返工。见下方"阶段二"末的「游戏机制知识库」和 `references/game-knowledge-base.md`。

---

## 阶段一：前置检查（每次会话开始先确认）

**为什么需要 .NET**：太吾后端跑在 .NET 8 运行时上（`Backend\GameData.runtimeconfig.json` 的 `tfm: net8.0`），所以**后端 mod 工程必须以 `net8.0` 为目标**，这就要求本机装 .NET 8 或更高。同时反编译工具 `ilspycmd` 是 framework-dependent 的 dotnet 全局工具，它依赖特定 .NET runtime 才能运行。**.NET 8 是最低要求**。

### 1a. 检查 .NET 开发环境

```bash
# 看最高 SDK 主版本（决定能 target 多高、装哪个 ilspycmd）
dotnet --list-sdks | grep -oE '^[0-9]+' | sort -n | tail -1
```

PowerShell 版（无 grep 时）：
```powershell
(dotnet --list-sdks) -replace '\..*','' | Sort-Object { [int]$_ } | Select-Object -Last 1
```

按最高 SDK 主版本（记为 `N`）判断：

| N | 状态 | 处理 |
|---|---|---|
| `≥ 8` | ✅ 满足最低要求 | 继续到 1b |
| `空` / 没装 / `< 8` | ❌ 缺失或过低 | 指引用户安装，**推荐装 .NET 10**（当前最新 LTS，向后兼容 net8.0）。下载地址 https://dotnet.microsoft.com/download 。装完新开终端再验。 |

> 推荐 .NET 10 而非 8：高版本 SDK 能编低版本目标（前向兼容），net10 的 SDK 既能编 `net8.0` 后端、也能编 `netstandard2.1` 前端，一步到位。且 ilspycmd 10.x 要求 net10 runtime。

### 1b. 检查并安装 ilspycmd（版本按 .NET 匹配）

`ilspycmd` 是反编译工具，但**它的版本和 .NET 版本有依赖关系**——每个大版本对应特定 .NET runtime（实测 ilspycmd 10.x 是 net10.0 目标）。装错版本会因 runtime 缺失报 "You must install .NET"。按上一步的 `N` 选版本：

```bash
ilspycmd --version   # 看是否已装
```

**未装时**，按本机最高 SDK 主版本 `N` 选对应 ilspycmd：

| 最高 SDK 主版本 N | 安装命令 |
|---|---|
| **10** | `dotnet tool install --global ilspycmd --version 10.*` |
| **8 或 9** | `dotnet tool install --global ilspycmd --version 9.*` |
| **> 10**（识别到更高版本，向后兼容） | 按默认最新版装：`dotnet tool install --global ilspycmd` |

> 逻辑：ilspycmd 10.x 要 net10 runtime、9.x 要 net8/9 runtime。装和本机最高 SDK 匹配的 ilspycmd 版本，确保它能运行。装完提示：全局工具目录需在 PATH（`dotnet tool install -g` 会提示路径，通常要新开终端才生效）。

### 1c. 定位游戏安装目录

**首选：从注册表读**（Steam 标准 uninstall key，可靠）。游戏 Steam App ID 是 **838350**：

```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Steam App 838350" /v InstallLocation
```

这个 key 的 `InstallLocation` 直接是游戏根目录（如 `D:\...\The Scroll Of Taiwu`），`DisplayName` 可二次确认是太吾。

> 该 key 在 `HKLM\SOFTWARE\` 的 64 位视图下，`reg query` 直接能读，**不要**加 `WOW6432Node`。
> 太吾还有 Unity 运行时 key `HKCU\Software\Conchship\The Scroll of Taiwu`，但它只存玩家偏好（分辨率等），**不含安装路径**，定位路径时不要用它。

**注册表读不到时**（非 Steam 安装、key 被清等）：让用户手动输入游戏根目录。确认目录下存在 `Backend\GameData.dll` 即为正确根目录。

### 1d. 游戏目录里有什么（开发要用到）

**关键：游戏有前后端两个进程，对应两个 DLL 目录。** 前后端 DLL 各自在自己进程的目录里，不要混用——用错目录会导致反编译/引用到错误的程序集（前端目录的 `GameData.Shared.dll` 和后端的 `GameData.dll` 不是同一个东西）。

```
<游戏根>/
├── The Scroll of Taiwu.exe
│
├── The Scroll Of Taiwu_Data/Managed/   ← 前端进程（Unity）加载的 DLL
│   ├── Assembly-CSharp.dll             ← 前端主程序集（UI/渲染/ModSystem/ModManager）
│   ├── Assembly-CSharp-firstpass.dll
│   ├── GameData.Shared.dll             ← 前端用的共享类型（注意：不是后端逻辑！）
│   ├── TaiwuModdingLib.dll             ← Mod 基类 TaiwuRemakePlugin（前端这份）
│   └── 0Harmony.dll                    ← HarmonyLib（前端这份）
│
└── Backend/                             ← 后端进程（独立）加载的 DLL
    ├── GameData.dll                    ← 后端主程序集（含 TaiwuDomain、DomainManager、所有域逻辑）
    ├── GameData.Shared.dll             ← 后端用的共享类型（大小与前端那份不同）
    ├── GameData.*.dll                  ← 后端按域拆分（ActionPlanning/Combat.Cricket/Adventure...）
    ├── TaiwuModdingLib.dll             ← Mod 基类（后端这份）
    └── 0Harmony.dll                    ← HarmonyLib（后端这份）
```

**端侧与目录的对应关系（务必记牢，最容易踩的坑）：**

| 要做什么 | 反编译 / 引用哪个目录的 DLL |
|---|---|
| 改后端逻辑（数值/规则/战斗/AI/事件） | **`Backend\`** 下的 `GameData.dll`（主）+ `GameData.*.dll` |
| 改前端行为（UI/渲染/输入/ModManager） | **`Managed\`** 下的 `Assembly-CSharp.dll` |
| 引用 `TaiwuRemakePlugin` 基类 | 用**目标端侧目录**那份 `TaiwuModdingLib.dll`（后端 mod 引 Backend 的，前端 mod 引 Managed 的） |
| 引用 Harmony | 同上，用目标端侧目录的 `0Harmony.dll` |

> ⚠️ **常见错误**：去 `Managed\` 找 `GameData.dll` 会找不到——后端主程序集只在 `Backend\`。`Managed\` 下只有 `GameData.Shared.dll`（共享类型库，不含 `TaiwuDomain` 等域逻辑），把它当后端主 dll 反编译会得到一份"缺了核心类"的源码。
>
> 2025.09"石牢三魔"版本把后端拆成多个 `GameData.*.dll`，后端逻辑按域分布在 `Backend\` 下，反编译/引用时用到哪个域加哪个。

---

## 阶段二：反编译源码就绪（阅读用，非编译用）

**反编译源码是用来读的**——查类名、方法签名、参数类型。编译 mod 引用的是游戏目录的真实 DLL（见阶段三）。这两者分开：源码读、DLL 引用。

### 2a. 版本一致性校验（关键）

游戏更新后签名会变，旧反编译源码会**误导**你写出对不上签名的 patch。每次要用源码前先校验版本是否一致。

**版本指纹**：Steam 的 `buildid`。读 `<游戏根>\..\appmanifest_838350.acf`（即 `steamapps/appmanifest_838350.acf`）里的 `buildid` 值。游戏每次更新 buildid 必变，是判断"反编译源码是否过期"的可靠锚点（和反编译目录路径里的 buildid 对比即可）。

> 注意区分两个"版本"：`buildid` 用于**判断反编译源码是否过期**；填进 Config.Lua 的 `GameVersion` 是**人类可读版本号**，从 `level0` 提取（见 config-lua-and-settings.md「怎么查当前游戏版本」）。两者用途不同，buildid 不能直接填进 GameVersion。

### 2b. 反编译缓存约定

所有反编译产物集中缓存在一个独立位置：`E:\taiwu_decompiled\`，按游戏 buildid 分目录，**一次制作，所有 mod 项目复用**。

```
E:\taiwu_decompiled\               ← 集中缓存根目录
├── decomp.json                    ← 元数据（buildid、时间、游戏版本）
└── <buildid>/
    ├── Assembly-CSharp/           ← 前端反编译源码（来自 Managed\Assembly-CSharp.dll）
    ├── GameData/                  ← 后端反编译源码（来自 Backend\GameData.dll）
    ├── GameData.Shared/           ← 共享类型库反编译（来自 Backend\GameData.Shared.dll）
    ├── knowledge-base/            ← 百晓册知识库（dotnet-build-kb 生成）
    └── config/                    ← 配置数值（config-extractor 生成）
```

> `E:\taiwu_decompiled\` 是跨所有 mod 项目共享的缓存目录，不入任何项目的 git。每个反编译周期会一次性重建全部内容（源码、知识库、配置），保持版本一致。

**元数据文件 `decomp.json`**（位于 `E:\taiwu_decompiled\` 根下）：

```json
{
  "buildid": "12345678",
  "last_updated": "2026-07-23 12:00",
  "game_version": "1.0.44"
}
```

| 字段 | 来源 | 用途 |
|------|------|------|
| `buildid` | Steam `appmanifest_838350.acf` | 核心判断：与当前游戏 buildid 对比，不一致则过期 |
| `last_updated` | 当前时间 | 辅助参考，如用户想了解上次反编译时间 |
| `game_version` | 从 `level0` 提取 | 填写 Config.Lua 的 `GameVersion` 时使用 |
### 2c. 缓存判断：三段式检查

每次需要反编译源码时，按以下顺序判断：

1. **读 `E:\taiwu_decompiled\decomp.json`** → 取出其中的 `buildid`。
2. **读 Steam `appmanifest_838350.acf`** → 取出当前游戏的 `buildid`。
3. **对比两个 buildid**：
   - **一致** → 跳过反编译，直接复用 `E:\taiwu_decompiled\<buildid>/` 下的全套文件。
   - **不一致** → 触发全量重建（一次性完成全部反编译 + 知识库 + 配置提取），见 2d。

> **任何情况下**，只要用户提到"游戏更新了"，或在开发过程中发现 buildid 与 `decomp.json` 不一致，都应当**立即触发全量重建**，不要犹豫。
>
> 反编译是**一次性全量制作**——前端（Assembly-CSharp）、后端（GameData）、共享类型库（GameData.Shared）、知识库、配置数值全部在同一周期完成。不要用到什么反编译什么。一次制作，所有会话、所有 mod 项目重复使用。

**额外校验**：复用前确认 `E:\taiwu_decompiled\<buildid>\GameData\` 里能找到核心类（如 `grep "class TaiwuDomain"` 命中）。若为空，说明之前反编译的目标错了（误用了 `GameData.Shared.dll`），需重建。
### 2d. 反编译命令

用 `-p`（生成可编译项目格式，便于阅读和 IDE 跳转）+ `-o`（输出目录）。**注意前端和后端主 dll 在不同目录**：

```powershell
# 变量
$GameDir = "<阶段一拿到的游戏根>"
$BuildId = "<appmanifest 的 buildid>"
$CacheRoot = "E:\taiwu_decompiled"          # 集中缓存根
$OutRoot = "$CacheRoot\$BuildId"

# 创建输出目录
New-Item -ItemType Directory -Force -Path "$OutRoot\Assembly-CSharp" | Out-Null
New-Item -ItemType Directory -Force -Path "$OutRoot\GameData" | Out-Null
New-Item -ItemType Directory -Force -Path "$OutRoot\GameData.Shared" | Out-Null

# 前端主程序集 → 在 Managed\
ilspycmd -p -o "$OutRoot\Assembly-CSharp" "$GameDir\The Scroll Of Taiwu_Data\Managed\Assembly-CSharp.dll"

# 后端主程序集 → 在 Backend\（不是 Managed\！Managed 下没有 GameData.dll）
ilspycmd -p -o "$OutRoot\GameData" "$GameDir\Backend\GameData.dll"

# 共享类型库 → 在 Backend\（含前后端通信用到的 SerializableModData 等）
ilspycmd -p -o "$OutRoot\GameData.Shared" "$GameDir\Backend\GameData.Shared.dll"
```

> 后端主 dll 路径务必是 `Backend\GameData.dll`。写成 `Managed\GameData.dll` 会因文件不存在失败，或（若历史版本结构不同）反编译成不含核心域逻辑的错误程序集。
> 反编译产物落在 `E:\taiwu_decompiled\` 而非工作区，跨会话、跨所有 mod 项目都可复用。

**反编译完成后，立即更新 `E:\taiwu_decompiled\decomp.json`**：

```powershell
# 从 level0 提取游戏版本号（方法见 config-lua-and-settings.md「怎么查当前游戏版本」）
$GameVersion = "<从 level0 提取的版本号，如 1.0.44>"

$decompJson = @"
{
  "buildid": "$BuildId",
  "last_updated": "$(Get-Date -Format 'yyyy-MM-dd HH:mm')",
  "game_version": "$GameVersion"
}
"@
$decompJson | Set-Content -Encoding UTF8 "$CacheRoot\decomp.json"
```

**然后构建知识库和配置数值**。详见下方「阶段二补充」及对应的 reference。

- **有旧 buildid 目录但无需清理**——重建会写进新 buildid 目录，旧版本保留可对照。
- 反编译耗时几分钟，进度会刷屏，正常。
### 2e. 阅读源码的姿势

把它们当源码读，不要当二进制猜——类名、方法名、参数签名都完整：
- **前端**（Unity 侧）：UI、渲染、`FrameWork.ModSystem`（Mod 加载与设置项）、`ModManager.cs`（游戏自己的 Mod 加载器）、`Game.Views.*`。
- **后端**：核心逻辑全在这，按 Domain 拆分：`GameData.Domains.Character`、`.Combat`、`.Item`、`.Map`、`.Organization`、`.TaiwuEvent` 等。
- 用 Grep 精确定位方法签名，记下 `类名.方法名(参数类型列表)`——Harmony patch 必须精确匹配。
- 遇到找不到的类型（如前后端通信用到的 `SerializableModData`），它很可能在 **`GameData.Shared.dll`**（前后端共享类型库），不在主 `GameData.dll`。单独反编译 `Backend\GameData.Shared.dll` 补充。

### 2f. 反编译第三方 mod 学习（强烈推荐）

社区已发布的 mod 是最好的实战教材——它们展示了游戏 API 的真实用法。遇到"不知道某个功能怎么做"时，先找一个实现了类似功能的 mod 反编译来看。

**第三方 mod 在哪**：Steam 订阅的 mod 在 `<Steam>\steamapps\workshop\content\838350\<mod的FileId>\`，结构和你自己开发的 mod 一样（`Config.Lua` + `Plugins\*.dll`）。FileId 在 mod 的 Config.Lua 或创意工坊页面 URL 里。

**反编译命令**（和反编译游戏 dll 一样，指向 mod 的 dll）：

```bash
# mod 的 dll 放到工作区 decompiled/mods/<ModName>/ 下
$ModDir = "<mod 目录>"
ilspycmd -p -o ".\decompiled\mods\<ModName>\Backend"  "$ModDir\Plugins\<后端dll>.dll"
ilspycmd -p -o ".\decompiled\mods\<ModName>\Frontend" "$ModDir\Plugins\<前端dll>.dll"
```

**优先挑带 `.pdb` 的 mod**——调试符号保留原始变量名，反编译质量远高于纯 dll（能看到真实命名而非 `num`/`val`）。判断方法：mod 的 `Plugins\` 目录下有同名 `.pdb` 文件。

**怎么读**：先读 `Config.Lua` 搞清这个 mod 的入口（前端/后端各几个 dll、有哪些设置项），再读每端的 `ModMain.cs`（或入口 Plugin 类）看 `Initialize` 里 patch 了什么、注册了什么 RPC，按需深入。

> 遇到 RPC / 前后端通信等不明白的问题时，可以参考一个生产级范例：创意工坊的「手动存档[天心帷幕正式版]」（FileId `2871612756`，带 .pdb，质量高）。先在本机 `<Steam>\steamapps\workshop\content\838350\2871612756\` 找有没有已订阅下载的；没有的话请用户去订阅一下再反编译。**没必要就别看——只在需要参考具体写法时才反编译**。

---

## 阶段二补充：游戏机制知识库（可选，推荐）

**反编译源码回答「代码怎么实现」，但回答不了「这个机制是什么、用什么数值、怎么运作」。** 游戏自带的《太吾百晓册》（官方百科）用文字详尽解释了大量游戏机制和数值——把它转成 markdown 知识库，让 AI 先理解机制、再结合源码定位实现，patch 写得更准、少返工。这在数值微调、规则修改类 mod 上尤其有用（你得先知道现状数值是多少，才知道改成什么）。

知识库由 skill 自带的 .NET 构建器从**当前安装的游戏资源**生成，是本地产物，与反编译源码一起集中缓存在 `E:\taiwu_decompiled\<buildid>/knowledge-base/`。完整指引（何时用、怎么构建、怎么查、两层各自用途、与源码的分工）见 `references/game-knowledge-base.md`。速记：

```bash
# 构建知识库，输出到 E:\taiwu_decompiled 缓存目录
# ⚠️ --project 必须用 skill 自身目录的【绝对路径】（即本 SKILL.md 所在目录），
#    不要写相对路径 scripts/...——scripts/ 在 skill 安装目录里，不在用户工作目录下。
dotnet run --project "<skill目录绝对路径>/scripts/dotnet-build-kb" -c Release -- -o "E:\taiwu_decompiled"
# 例（skill 装在 .agents/skills/taiwu-mod-dev/ 时）：
# dotnet run --project "D:/my-mod/.agents/skills/taiwu-mod-dev/scripts/dotnet-build-kb" -c Release
```

- 构建器工程在 skill 包内的 `scripts/dotnet-build-kb/`（零 NuGet 依赖，`dotnet run` 现场编译）；知识库与反编译源码一起输出到 `E:\taiwu_decompiled\<buildid>/knowledge-base/`。
- 生成两层：① 百晓册正文（机制解释，按顶级章节分文件）② 数据表（表头已 JOIN 还原）。秒级生成。
- 用同一个 **buildid 锚点**和反编译源码保持一致：游戏更新后 buildid 变，知识库和源码要么都最新、要么都重建。
- **查询姿势**：先读 `E:\taiwu_decompiled\<buildid>/knowledge-base/INDEX.md` 定向，再深读对应文件；机制问题查正文、数值问题查数据表。
- **配合源码**：用户要做"免疫中毒"→ 先查知识库正文「人物>伤病>毒素」理解中毒机制，再反编译 Grep 毒素方法签名写 patch。

### 游戏配置数值（config-extractor）

百晓册答「机制怎么运作」，但**具体数值**（某特性加多少属性、某功法破体破气多少、某武器破甲多少）要从 `Backend\GameData.Shared.dll` 提取——这些数值硬编码在 IL 里，不在百晓册明文。config-extractor 用 Mono.Cecil 静态解析 IL，离线提取全部配置表的真实数值。**做数值微调类 mod 时尤其关键**（先查现状数值，才知道 patch 成什么）。完整指引见 `references/game-config.md`。速记：

```bash
# 构建配置数值，输出到 E:\taiwu_decompiled 缓存目录
dotnet run --project "<skill目录绝对路径>/scripts/config-extractor" -c Release -- -o "E:\taiwu_decompiled"
```

- 产物落在 `E:\taiwu_decompiled\<buildid>/config/`（每张表一个 JSON，含全部实体的完整字段数值），与 `knowledge-base/` 同 buildid 锚点。
- **查询**：先看 `E:\taiwu_decompiled\<buildid>/config/_manifest.json` 找目标表（如 `CharacterFeature`/`CombatSkill`/`Weapon`），再读对应 JSON 按 `TemplateId`/`Name` 定位实体。
- 字段名是英文 PascalCase，需结合百晓册正文或反编译对应 `Config.*Item` 类理解字段含义。

---

## 阶段三：开发

### 3a. 插件入口契约（最小骨架）

后端插件示例：

```csharp
using TaiwuModdingLib.Core.Plugin;

namespace MyMod.Backend;

[PluginConfig("MyMod.Unique.Backend", "<用户确认的作者名>", "1.0.0")]
public sealed class BackendPlugin : TaiwuRemakePlugin
{
    public override void Initialize()
    {
        // ModIdStr 是基类字段：当前 mod 运行时标识，打 Harmony id / 读设置都用它
        MyPatches.Install(ModIdStr);
    }

    public override void Dispose() => MyPatches.Uninstall();

    public override void OnModSettingUpdate() { }  // 可选
}
```

- `[PluginConfig]` 三参：**全局唯一标识符**（带命名空间防冲突）、作者、版本。**作者名不要编**——按下方 3c「作者名怎么定」从 Steam 昵称探测 + 用户确认得来。
- `TaiwuRemakePlugin` 基类来自游戏自带的 `TaiwuModdingLib.dll`。
- `Initialize`/`Dispose` 是生命周期钩子；`OnModSettingUpdate` 仅在 `NeedRestartWhenSettingChanged=false` 时有意义。

### 3b. 工作区 .gitignore（创建项目时自动生成）

在 mod 项目工作区根目录生成 `.gitignore`，保证只跟踪必要文件：

```
# 只保留：src/（源码）、mod/（最终 mod）、README.md
# 其余全部过滤

decompiled/
other/
bin/
obj/
packages/
*.nupkg
*.user
*.suo
*.DotSettings
.vs/
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store
Thumbs.db
desktop.ini
mod/Settings.Lua
```

> `src/` 是源代码，`mod/` 是最终 mod 文件（直接复制到游戏 Mod/ 就能用），这两项和 `README.md` 一起入库，其余编译产物、反编译参考、IDE 配置、系统文件统统不入库。`mod/Settings.Lua` 由游戏运行时自动生成，也不入库。

### 3c. Config.Lua（必备，缺了 mod 加载不了）


```lua
return {
  Title = "我的 Mod",
  Version = "1.0.0",
  GameVersion = "1.0.44",            -- 当前游戏版本；获取方法见 config-lua-and-settings.md「怎么查当前游戏版本」
  Author = "<用户确认的作者名>",
  Description = "支持 [h1][b][list] 等 Steam BBCode",
  BackendPlugins = { "MyMod.Backend.dll" },   -- dll 相对 Plugins/ 的路径，用 ../ 可跳出
  DefaultSettings = {                          -- 可选设置项
    { SettingType = "Toggle", Key = "EnableX", DisplayName = "启用", Description = "", DefaultValue = true },
  },
  NeedRestartWhenSettingChanged = true,
}
```

设置项类型（Toggle/InputField/Slider/Dropdown/ToggleGroup）与读写 API 见 `references/config-lua-and-settings.md`。

### 作者名怎么定（重要，不要编）

> ⚠️ **绝不能把 `Author` 留占位符或自己编一个名字**（如 `taiwu-mod-dev`）——那不是 mod 开发者想要的。`Author` 必须来自用户：**先自动探测最近登录的 Steam 昵称作候选，再用 AskUserQuestion 让用户确认或输入**。完整脚本和交互方式见 `references/config-lua-and-settings.md`「确定 Author（作者名）」。流程速记：
> 1. **探测**：PowerShell 读 `$(HKCU:\Software\Valve\Steam SteamPath)\config\loginusers.vdf`，取 `PersonaName`（优先 `MostRecent=1`，否则 `Timestamp` 最新）。
> 2. **成功** → AskUserQuestion：把昵称作为推荐项，用户确认 / 自定义 / 留空。
> 3. **失败**（无 Steam/无 vdf/没登录过）→ 直接 AskUserQuestion 让用户输入，不给占位默认值。
> 确定后填进 `Config.Lua` 的 `Author` 和 `[PluginConfig]` 第二参（两处一致）。

### 3d. Harmony patch、工程搭建、编译部署

- **打后端 patch（改数值/规则/战斗/AI/事件）** → 先读 `references/backend-harmony.md`（签名匹配、Prefix/Postfix/Transpiler、DataContext、判断太吾身份）。
- **建可编译的独立工程、引用哪个游戏 DLL、打包部署到游戏目录** → 先读 `references/project-setup.md`（包含从游戏目录拷贝 dll 的做法）。
- **改前端 / UI** → `references/frontend-notes.md`。
- **跨端 mod（前端要调用你新加的后端方法）** → 必读 `references/frontend-backend-rpc.md`（RPC 机制、`SerializableModData` 数据载体、四种调用变体）。

### 3e. 部署目录结构

```
<游戏>/Mod/MyMod/
├── Config.Lua          ← 必备
├── Settings.Lua        ← 可选，玩家改设置后自动生成
├── Cover.png           ← 可选
└── Plugins/
    ├── MyMod.Backend.dll
    └── (依赖 dll)
```

### 3f. 调试：日志

- `%USERPROFILE%\AppData\LocalLow\Conchship\The Scroll of Taiwu\Player.log`（主日志；`Player-prev.log` 是上次启动）
- 游戏文件夹 `Logs/`
- 前端用 `UnityEngine.Debug.Log`；patch 不生效、mod 没加载、运行时报错，先翻这两个日志。
- 当前游戏版本号：从启动场景 `level0` 离线提取（不需要启动游戏），见 config-lua-and-settings.md「怎么查当前游戏版本」；提取失败则请用户启动一次游戏后从 Player.log 读。

---

## 阶段四：发布（到 Steam 创意工坊）

mod 开发完成、测试通过后，发布到创意工坊。**发布前必须先完善 Config.Lua 并做自检**——这是本 skill 的明确要求。完整指引见 `references/publishing.md`。

### 4a. 发布前：完善 Config.Lua（与用户交互）

发布前 Config.Lua 不能停留在开发占位状态。**用 AskUserQuestion 收集必要信息**：

- **Title**（必填，不能空、不能与本地其他 mod 重名）
- **Author**（作者）。**不要自己编，也不要留占位符**：先按 3c「作者名怎么定」自动探测最近登录的 Steam 昵称，探测到了用 AskUserQuestion 让用户确认（昵称作为推荐项，可选自定义/留空）；探测失败直接让用户输入。
- **Version**（必填，更新时必须递增）
- **GameVersion**（用当前游戏版本。从 `level0` 离线提取，方法见 `references/config-lua-and-settings.md`「怎么查当前游戏版本」；提取失败则请用户启动一次游戏后从 Player.log 读）
- 推荐还收集：Description（BBCode 描述）、TagList（工坊标签）、封面图

`FileId`/`Source`/`UpdateLogList` **不要手填**——发布时游戏会自动管理。

### 4b. 发布前自检

逐项确认（详见 publishing.md 的完整清单），关键几条：
- Config.Lua 字段完整、Version 递增、GameVersion 匹配当前游戏。
- dll 已部署到 `Plugins\`，且 **`Plugins\` 里没有多余的游戏 dll**（`<Private>false</Private>`）。
- mod 目录干净，无 `.cs`/`.csproj`/`refs\`/`bin\`/`obj\` 等开发文件。
- 删掉 `Settings.Lua`（让玩家用默认设置）。

### 4c. 在游戏内发布

发布由**游戏内 mod 管理器 UI**完成（`ModUploadEditPanel`），不要用脚本绕开：
1. 启动太吾 → Mod 管理器 → 选你的 mod → 进上传编辑面板。
2. 确认信息 → 首次点"上传"/更新点"更新" → 填更新日志 → 等上传完成。
3. 成功后 `Config.Lua` 自动写入 `FileId`（创意工坊 ID），后续更新靠它定位。

### 4d. 更新与维护（已发布的 mod）

mod 发布后还会持续迭代——要么修 bug/加功能，要么适配游戏新版本。这是和"首次发布"不同的场景，**完整引导见 `references/publishing.md` 的「更新已发布的 mod」和「游戏更新后如何维护 mod」**。关键点速记：

- **每次发布新版都要递增 `Version`**（Steam 靠它判定有更新；版本号只能单调递增，无法回退）。
- **`FileId` 不能动**——更新就是往同一个 FileId 推新内容，改了会变成新建 mod。
- **游戏更新后不一定影响你的 mod**：先跑缓存校验（见 2c），buildid 变了就全量重建到新目录，旧的保留在 `E:\taiwu_decompiled/`。直接对比新旧两个版本的源码——方法签名变了、改名了、类结构调整了，一目了然。确认没影响就不动，确定了再修复 + 递增 Version + 更新 `GameVersion`。
- **回滚靠 git 备份**：创意工坊不保留历史，建议 mod 工程用 git 管、每次发布打 tag。

## 核心原则

- **读源码定位，别靠猜。** 任何 patch 写之前先 Grep 到目标方法真实签名（含参数类型），否则 Harmony 静默失败。
- **源码读、DLL 引用，二者分开。** 反编译产物只读；编译必引用游戏目录的真实 DLL（前端 `Managed\`、后端 `Backend\`），且 `<Private>false</Private>` 防止类型身份冲突。
- **优先 public 方法、优先改配置/事件脚本。** 太吾大量内容是 lua 配置和事件脚本，改它们比代码 patch 稳。能用配置就别上 Harmony。
- **端侧分清，目录别混。** 前端 mod 引 `Managed\Assembly-CSharp.dll`，后端 mod 引 `Backend\GameData.dll`；跨端访问数据走 RPC（见 frontend-backend-rpc.md），不能直接互引对方端侧的主 dll。
- **TFM 按端侧选。** 后端工程 `net8.0`（后端跑 .NET 8）、前端工程 `netstandard2.1`（Unity Mono），不能混用。需要 .NET 8+ SDK，推荐 .NET 10（见阶段一 1a）。
- **版本敏感。** 游戏更新换签名后 patch 失效，需重新反编译对齐。
- **理解机制先查百晓册、查数值用 config、定位代码再反编译。** 三者用同一个 buildid 锚点保持一致：百晓册知识库（`E:\taiwu_decompiled/<buildid>/knowledge-base/`）答「机制/规则是什么」，config-extractor（`E:\taiwu_decompiled/<buildid>/config/`）答「每个实体的具体数值」，反编译源码答「代码怎么实现、方法签名」。分别见 `references/game-knowledge-base.md`、`references/game-config.md`。

## reference 何时读哪个

- `references/project-setup.md` — 建独立可编译工程 / 引用游戏 DLL（含拷贝 dll） / 打包部署（含 Release 编译后自动部署到游戏 Mod 目录）。**第一次搭工程必读。**
- `references/frontend-backend-rpc.md` — 跨端 mod（前端调用你用 `AddModMethod` 新加的后端方法）：RPC 机制、`SerializableModData` 数据载体、四种调用变体、ModId 同步。**做跨端 mod 必读。**
- `references/backend-harmony.md` — 后端 patch 全套（签名、Prefix/Postfix/Transpiler、DataContext、判断太吾身份）。
- `references/publishing.md` — 发布到创意工坊：发布前 Config.Lua 必填字段与交互、自检清单、游戏内发布步骤、版本号格式、**更新已发布 mod 的完整工作流、游戏更新后如何维护/适配 mod、回滚**。**发布前及每次更新必读。**
- `references/config-lua-and-settings.md` — Config.Lua 全字段 + 设置项 5 种类型 + 读写 API。
- `references/frontend-notes.md` — 前端 / UI mod 要点。
- `references/game-knowledge-base.md` — 游戏机制知识库（百晓册）：何时用、怎么用 .NET 构建器生成、怎么查、两层（正文/数据表）各自用途、与反编译源码的分工。**需要理解游戏机制/规则时读。**
- `references/game-config.md` — 游戏配置数值（config-extractor）：从 GameData.Shared.dll 离线提取全部配置表的真实数值（特性/武器/功法等每个实体的完整字段）。**做数值微调、需要查某实体当前数值时读。**
