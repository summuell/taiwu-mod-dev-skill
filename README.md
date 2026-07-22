<div align="center">

# 澶惥缁樺嵎 Mod 寮€鍙?.skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-Standard-green)](https://github.com/vercel-labs/skills)
[![skills.sh](https://img.shields.io/badge/skills.sh-Compatible-blue)](https://skills.sh)
[![Multi-Runtime](https://img.shields.io/badge/Runtime-Claude%20Code%20%C2%B7%20Codex%20%C2%B7%20Cursor%20%C2%B7%20ZCode-blueviolet)](#瀹夎)

<br>

**涓恒€婂お鍚剧粯鍗凤細澶╁箷蹇冨阜銆嬶紙The Scroll of Taiwu锛孲team App 838350锛夊埗浣滅嫭绔?C# Mod 鐨勫叏娴佺▼ skill銆?*

<br>

浠庣幆澧冩鏌ワ紙.NET SDK銆乮lspycmd銆佹父鎴忓畾浣嶏級銆佹寜闇€鍙嶇紪璇戞父鎴忕▼搴忛泦銆佺増鏈竴鑷存€ф牎楠岋紝
鍒扮紪鍐欐彃浠跺叆鍙ｃ€佺敤 HarmonyLib 鎵撹ˉ涓併€佸畾涔?Config.Lua 璁剧疆椤广€佸墠鍚庣 RPC 閫氫俊锛?鍐嶅埌缂栬瘧閮ㄧ讲銆佺湅鏃ュ織璋冭瘯銆佸彂甯冨埌鍒涙剰宸ュ潑鍙婂悗缁増鏈淮鎶も€斺€?*瀹屾暣閾捐矾锛屼竴涓?skill 鎼炲畾**銆?
杩樺唴缃?*娓告垙鏈哄埗鐭ヨ瘑搴?*锛氭妸娓告垙鍐呫€婂お鍚剧櫨鏅撳唽銆嬶紙瀹樻柟鐧剧锛夎浆鎴?markdown锛岃 AI 鍏堢悊瑙ｆ父鎴忔満鍒?鏁板€煎啀缁撳悎鍙嶇紪璇戞簮鐮佸畾浣嶅疄鐜帮紝patch 鍐欏緱鏇村噯銆?

[瀹夎](#瀹夎) 路 [杩欎釜 skill 鑳藉仛浠€涔圿(#杩欎釜-skill-鑳藉仛浠€涔? 路 [宸ヤ綔娴乚(#宸ヤ綔娴? 路 [璁捐瑕佺偣](#璁捐瑕佺偣)


> 本仓库改编自 [gruiyuan/taiwu-mod-dev-skill](https://github.com/gruiyuan/taiwu-mod-dev-skill)，在原版基础上调整了反编译缓存策略和工作区结构。感谢原作者的出色工作。

</div>

---

## 瀹夎

鏀寔涓夌鏂瑰紡锛屼换閫夊叾涓€銆?
### 鏂瑰紡涓€锛氭妸鏈粨搴撻摼鎺ヨ创缁?AI Agent锛堟渶鐪佷簨锛?
鐩存帴鎶婁笅闈㈣繖琛屽彂缁欎綘鐨?AI Agent锛圕laude Code / Codex / Cursor / ZCode 绛夋敮鎸?Agent Skills 鏍囧噯鐨勫鎴风锛夛細

```
璇峰畨瑁呰繖涓?skill锛歨ttps://github.com/summuell/taiwu-mod-dev-skill
```

Agent 浼氳嚜鍔ㄨ瘑鍒苟瀹夎锛屼箣鍚庝綘鐩存帴璇淬€屽府鎴戝仛涓お鍚剧殑 mod銆嶅嵆鍙Е鍙戙€?
### 鏂瑰紡浜岋細鍛戒护瀹夎锛堟帹鑽愶級

浣跨敤 [vercel-labs/skills](https://github.com/vercel-labs/skills) 鐨?CLI 涓€閿畨瑁呫€?*鍦ㄤ綘鐨勫お鍚?mod 宸ョ▼鐩綍涓嬫墽琛?*鈥斺€旀湰 skill 鏄お鍚句笓鐢ㄣ€佷笉閫傚悎鏀惧叏灞€锛岄粯璁ゅ氨瑁呭埌褰撳墠椤圭洰锛?
```bash
npx skills add summuell/taiwu-mod-dev-skill
```

> 璇ュ懡浠ら粯璁よ鍒?*褰撳墠椤圭洰鐨?* agent skills 鐩綍锛堥」鐩骇锛屼細闅忎粨搴撴彁浜ゅ叡浜級銆?*涓嶈鍔?`-g`/`--global`**鈥斺€旈偅鏄鍒板叏灞€鐢ㄦ埛鐩綍锛屼細璁╂墍鏈夐」鐩兘鍔犺浇杩欎釜澶惥涓撶敤 skill锛屾病鏈夊繀瑕併€?> 涔熷吋瀹?`npx skill add summuell/taiwu-mod-dev-skill` 鍐欐硶銆?
### 鏂瑰紡涓夛細鎵嬪姩瀹夎

鎶婃湰浠撳簱鐨?`SKILL.md` 鍜?`references/` 鐩綍澶嶅埗鍒颁綘鐨?Agent skills 鐩綍鍗冲彲锛?
```bash
git clone https://github.com/summuell/taiwu-mod-dev-skill.git
```

鐒跺悗鏍规嵁浣犵敤鐨?Agent锛屾斁鍒板搴斾綅缃紙浠婚€夊叾涓€锛夛細

鐒跺悗鍦ㄤ綘鐨勫お鍚?mod 宸ョ▼鏍圭洰褰曚笅锛屾寜鎵€鐢ㄧ殑 Agent 鏀惧埌瀵瑰簲椤圭洰绾х洰褰曪紙鏈?skill 涓嶅缓璁斁鍏ㄥ眬锛夛細

| Agent | 椤圭洰绾?skills 鐩綍 |
|---|---|
| Claude Code | `<椤圭洰>/.claude/skills/taiwu-mod-dev/` |
| Codex | `<椤圭洰>/.codex/skills/taiwu-mod-dev/` |
| Cursor | `<椤圭洰>/.cursor/skills/taiwu-mod-dev/` |
| OpenCode | `<椤圭洰>/.opencode/skills/taiwu-mod-dev/` |
| ZCode | `<椤圭洰>/.agents/skills/taiwu-mod-dev/` |

> 鏈?skill 鏄お鍚句笓鐢ㄣ€佷笉閫傚悎鏀惧叏灞€锛堜細璁╂墍鏈夐」鐩兘鍔犺浇锛夈€傚悇 Agent 閮芥敮鎸侀」鐩骇瀹夎锛屾瘡涓敤鑷繁鐨勭偣鐩綍锛坄.claude` / `.codex` / `.cursor` / `.opencode` / ZCode 鐢?`.agents`锛夈€俙<椤圭洰>` 鎸囦綘鐨勫お鍚?mod 宸ョ▼鏍圭洰褰曘€?>
> Cursor 娉ㄦ剰锛歚.cursor/skills-cursor/` 鏄?Cursor 鍐呯疆鍙鐩綍锛岀敤鎴?skill 鏀?`.cursor/skills/`锛堟棤 `-cursor` 鍚庣紑锛夈€?
鐩綍缁撴瀯搴斾负锛?
```
taiwu-mod-dev/
鈹溾攢鈹€ SKILL.md
鈹溾攢鈹€ scripts/
鈹?  鈹溾攢鈹€ dotnet-build-kb/            # 鐢熸垚銆婂お鍚剧櫨鏅撳唽銆嬫満鍒剁煡璇嗗簱鐨?.NET 宸ョ▼锛坉otnet run锛?鈹?  鈹斺攢鈹€ config-extractor/           # 浠庢父鎴?dll 绂荤嚎鎻愬彇鍏ㄩ儴閰嶇疆鏁板€肩殑 .NET 宸ョ▼锛圡ono.Cecil锛?鈹斺攢鈹€ references/
    鈹溾攢鈹€ backend-harmony.md
    鈹溾攢鈹€ config-lua-and-settings.md
    鈹溾攢鈹€ frontend-backend-rpc.md
    鈹溾攢鈹€ frontend-notes.md
    鈹溾攢鈹€ game-config.md             # 娓告垙閰嶇疆鏁板€硷紙config-extractor锛夌殑鏋勫缓涓庝娇鐢?    鈹溾攢鈹€ game-knowledge-base.md     # 娓告垙鏈哄埗鐭ヨ瘑搴擄紙鐧炬檽鍐岋級鐨勬瀯寤轰笌浣跨敤
    鈹溾攢鈹€ project-setup.md
    鈹斺攢鈹€ publishing.md
```

---

## 杩欎釜 skill 鑳藉仛浠€涔?
褰撲綘璇村嚭涓嬮潰杩欎簺锛宻kill 浼氳嚜鍔ㄦ帴绠★細

- 銆屾垜鎯冲仛涓澶惥鍏嶇柅涓瘨鐨?mod銆?- 銆屽府鎴戝姞涓墿鍝?/ 鏀规垬鏂椾激瀹炽€?- 銆屾垜鐨?mod 鍔犺浇涓嶄簡 / Harmony patch 涓嶇敓鏁堛€?- 銆屾€庝箞鍙嶇紪璇戝お鍚剧湅鐪嬫煇涓柟娉曠殑绛惧悕銆?- 銆屾€庝箞鏇存柊宸插彂甯冪殑 mod銆?- 銆屾父鎴忔洿鏂颁簡鎴戠殑 mod 杩樿兘鐢ㄥ悧銆?- 銆屽府鎴戝仛涓墠鍚庣閫氫俊鐨?mod銆?- 銆岃繖涓父鎴忔満鍒舵槸鎬庝箞璁捐鐨勶紵鏌愰」鏁板€兼槸澶氬皯锛熴€?
## 宸ヤ綔娴?
skill 鎶?mod 寮€鍙戠粍缁囨垚鍥涗釜闃舵锛屾瘡娆′細璇濇寜闇€鎺ㄨ繘锛?
1. **鍓嶇疆妫€鏌?* 鈥?.NET 8+ 鐜銆乮lspycmd 鐗堟湰鍖归厤銆佷粠娉ㄥ唽琛ㄥ畾浣嶆父鎴忓畨瑁呯洰褰曘€?2. **鍙嶇紪璇戝氨缁?* 鈥?鎸?Steam buildid 鍋氱増鏈竴鑷存€ф牎楠岋紝鎸夐渶鍙嶇紪璇戝墠绔?`Assembly-CSharp.dll` / 鍚庣 `GameData.dll` 鍒板伐浣滃尯銆傦紙鍙€?鎺ㄨ崘锛氬悓姝ョ敓鎴愩€婂お鍚剧櫨鏅撳唽銆嬫満鍒剁煡璇嗗簱銆傦級
3. **寮€鍙?* 鈥?鎻掍欢鍏ュ彛锛坄TaiwuRemakePlugin`锛夈€丠armony patch銆丆onfig.Lua 璁剧疆椤广€佸墠鍚庣 RPC锛岀紪璇戦儴缃茬湅鏃ュ織銆?4. **鍙戝竷涓庣淮鎶?* 鈥?瀹屽杽 Config.Lua锛堜氦浜掓敹闆嗕綔鑰?鐗堟湰绛変俊鎭級銆佽嚜妫€銆佹父鎴忓唴涓婁紶鍒涙剰宸ュ潑锛屼互鍙婄増鏈洿鏂般€侀€傞厤娓告垙鏂扮増鏈€佸洖婊氥€?
## 璁捐瑕佺偣

- **婧愮爜璇汇€丏LL 寮曠敤鍒嗗紑** 鈥?鍙嶇紪璇戜骇鐗╁彧璇伙紱缂栬瘧寮曠敤娓告垙鐩綍鐨勭湡瀹?DLL锛屼笖 `<Private>false</Private>` 闃茬被鍨嬭韩浠藉啿绐併€?- **鍓嶅悗绔洰褰曞埆娣?* 鈥?鍓嶇 `Managed\Assembly-CSharp.dll`銆佸悗绔?`Backend\GameData.dll`锛涘悗绔伐绋?`net8.0`銆佸墠绔伐绋?`netstandard2.1`銆?- **鐗堟湰涓嶉潬鐚?* 鈥?娓告垙鐗堟湰鍙蜂粠鍚姩鍦烘櫙 `level0` 绂荤嚎鎻愬彇锛堟棤闇€鍚姩娓告垙锛夛紱Steam `buildid` 鍒ゆ柇鍙嶇紪璇戞簮鐮佹槸鍚﹁繃鏈熴€?- **浼樺厛 public銆佷紭鍏堟敼閰嶇疆** 鈥?鑳芥敼 lua 閰嶇疆/浜嬩欢鑴氭湰灏卞埆涓?Harmony锛岃兘 patch public 灏卞埆纰?internal銆?
## 閫傜敤鐗堟湰

闈㈠悜褰撳墠銆婂お鍚剧粯鍗凤細澶╁箷蹇冨阜銆嬶紙鍚庣 .NET 8銆乁nity Mono 鍓嶇锛夈€傛父鎴忓ぇ鐗堟湰鏇存柊鍚庯紝鎸?skill 闃舵浜岀殑鐗堟湰涓€鑷存€ф牎楠岄噸鏂板弽缂栬瘧瀵归綈鍗冲彲銆?
## License

[MIT](LICENSE)

