# <ModName>

<ModName> —— 为《太吾绘卷：天幕心阜》（The Scroll of Taiwu，Steam App 838350）制作的独立 C# Mod。

---

## 目录结构

```
<ModName>/
├── mod/                       ← 最终 mod 文件，直接复制到游戏 Mod/ 即可使用
│   ├── Config.Lua             ← mod 元信息与可调设置项
│   ├── Settings.Lua           ← 玩家保存的设置（自动生成，不入库）
│   └── Plugins/               ← 编译产物的 DLL
│       └── ModName.Backend.dll
├── src/                       ← 源代码
│   ├── ModName.Backend/
│   │   ├── ModName.Backend.csproj
│   │   ├── BackendPlugin.cs   ← 插件入口（TaiwuRemakePlugin）
│   │   └── Patches/           ← Harmony patch 文件
│   └── ModName.Frontend/      ← 前端 mod（可选）
├── decompiled/mods/<ModName>/ ← （可选）参考第三方 mod 时反编译产物
└── other/                     ← 中间产物、临时脚本、笔记等
```

> `mod/` 目录是完整的可发布包，可直接复制到 `<游戏根目录>/Mod/<ModName>/` 使用。
> `src/` 下的 `.cs` / `.csproj` 仅用于开发和编译，不进入最终发布包。
> `decompiled/` 和 `other/` 建议加入 `.gitignore`。

---

## 前置条件

| 项目 | 要求 | 说明 |
|------|------|------|
| .NET SDK | >= 8.0（推荐 10.0） | 后端 mod 目标 `net8.0`，高版本 SDK 可向下编译 |
| ilspycmd | 版本匹配 .NET SDK | 用于反编译游戏代码查阅签名 |
| 太吾绘卷 | Steam 已安装 | 通过注册表自动定位安装目录 |

相关工具检查与安装流程参见 skill 的阶段一。

---

## 快速开始

### 1. 编译

```powershell
dotnet build src\ModName.Backend\ModName.Backend.csproj -c Release
```

编译产物自动输出到 `mod/Plugins/`。

### 2. 部署到游戏

```powershell
$GameDir = (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Steam App 838350').InstallLocation
Copy-Item 'mod\*' "$GameDir\Mod\<ModName>" -Recurse -Force
```

### 3. 查看日志

```
%USERPROFILE%\AppData\LocalLow\Conchship\The Scroll of Taiwu\Player.log
```

### 4. 发布到创意工坊

1. 确认 `Config.Lua` 的 `Author`、`Version`、`GameVersion` 已正确填写
2. 运行自检：`mod/` 目录下无 `.cs`、`.csproj`、`refs/` 等开发文件
3. 启动游戏 -> Mod 管理器 -> 选中本 Mod -> 进入上传编辑面板 -> 填写更新日志 -> 上传

详细流程参见 skill 的 `references/publishing.md`。

---

## 设计原则

- **源代码仅用于阅读，编译引用游戏目录的真实 DLL**
  反编译产物在 `E:\taiwu_decompiled\` 缓存目录中（跨项目复用、不入库），编译时通过 `<HintPath>` 直接引用游戏目录的 DLL，并设置 `<Private>false</Private>` 防止类型身份冲突。

- **优先改 Lua 配置 / 事件脚本，其次 Harmony patch**
  太吾大量内容是 Lua 配置和事件脚本，改它们比代码 patch 稳妥。

- **前后端分明，目录别混**
  后端 mod 引用 `Backend\GameData.dll`（目标 `net8.0`），前端 mod 引用 `Managed\Assembly-CSharp.dll`（目标 `netstandard2.1`）。跨端访问走 RPC。

- **版本敏感**
  游戏更新换签名后 Harmony patch 会静默失效，需重新反编译对账。版本一致性通过 Steam `buildid` 校验。

---

## 相关文档

| 文档 | 场景 |
|------|------|
| project-setup.md | 搭建独立可编译工程、引用 DLL、打包部署 |
| backend-harmony.md | 后端 Harmony patch 全套路（签名、Prefix/Postfix、DataContext） |
| frontend-notes.md | 前端 / UI mod 要点 |
| frontend-backend-rpc.md | 跨端 mod：RPC 机制、数据载体、四种调用变体 |
| config-lua-and-settings.md | Config.Lua 全字段 + 设置项 5 种类型 + 读写 API |
| publishing.md | 发布到创意工坊：必填字段、自检清单、版本号格式、更新与维护 |
| game-knowledge-base.md | 游戏机制知识库（百科），理解规则时查阅 |
| game-config.md | 游戏配置数值（config-extractor），做数值微调时查阅 |

---

## 兼容性

本 Mod 面向《太吾绘卷：天幕心阜》当前版本（后端 .NET 8、Unity Mono 前端）。

游戏大版本更新后，如 Mod 功能失效：
1. 检查 `E:\taiwu_decompiled\decomp.json` 的 buildid 是否与当前游戏一致
2. 重新反编译并对账 patch 过的签名
3. 若签名未变则无需改动
4. 如有变化则修复 + 递增 `Version` + 更新 `GameVersion`

---

## License

[MIT](LICENSE) — 详见仓库根目录的 LICENSE 文件。

---

> 本 Mod 由 [taiwu-mod-dev-skill](https://github.com/summuell/taiwu-mod-dev-skill) 辅助开发。
