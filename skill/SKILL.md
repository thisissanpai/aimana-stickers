---
name: sticker
description: 给 AI Mana 的对话回复配表情包。当回复出现情绪落点（任务完成、踩坑、认错、被指出问题、催办、自嘲、报喜）需要配图时使用；用户说"来张表情包/配个图/发表情/带个图"时也走这里。表情包库与索引位置由本技能指定，不要在回复中临时找图或现场生成图。
---

# 回复表情包

给对话回复配上表情包。**库和索引的位置由本技能指定，不要临时找图、不要现场生成图。**

## 先判断你的角色

本技能有两种用法，**先看你是哪种——这决定哪些章节跟你有关系**：

| 角色 | 怎么判断 | 要做什么 |
|---|---|---|
| **库主** | 机器上存在 `C:\Users\Admin\Documents\Stickers\` 目录 | 全部章节。加图看「加新图」，改完看「推送仓库」 |
| **使用者** | 没有那个目录（通常是同事） | **只读索引、显示图片。** 跳过「加新图」「推送仓库」 |

**使用者不需要装 git、不需要配密钥、不需要下载图片**——索引和图片都从公开仓库直接加载，装完即用。

## 使用频率

默认 **活泼**：几乎每轮回复都带一张。

**例外——以下情况不加图：**
- 正式交付物、成批数据结果、严肃排查结论、错误报告
- 用户明确要求"正式 / 严谨"，或正在处理严肃事务
- 用户正在表达**真实**挫败或焦虑时，不要用自嘲型表情包（会显得轻慢）

## 摆放规则

- **结论先行，表情包收尾。** 信息永远在文字里，表情包只负责语气，不能替代内容。
- **一轮最多 1 张**，不刷屏。
- 放在回复最后一个自然段之后，单独成行。

## 怎么选图

1. 读索引（位置见下方"索引位置"）。
2. 按当前语境的**情绪**匹配 `适用` 字段；命中 `禁忌` 的一律不用。
3. 没有合适的就**不加**——不要硬凑，不要连续两轮用同一张。

## 怎么输出

索引里每条有 `file`，拼上 base 写成 Markdown 图片。索引的 `base` 段给了两个位置：

```json
"base": { "local": "C:/Users/Admin/Documents/Stickers", "remote": "https://raw.githubusercontent.com/..." }
```

**选哪个：先试 `local`。**
- `local` 下该文件存在 → 用 `file:///` + 路径（**正斜杠**），快、可离线
- 不存在 → 拼 `remote` 的 URL

同事机器上没有那个本地目录，会自动落到 `remote`，不需要任何配置。这就是同一份索引能同时给本人和同事用的原因。

```
![名称](<base>/<file>)
```

> ⚠️ 路径**必须用正斜杠**。反斜杠在 Markdown 里是转义符，路径含 `\_`、`\*` 会被吃掉，表现为裂图。

## 尺寸：靠图片本身控制，不靠语法

**已验证的渲染能力（2026-10-08 实测于 AI Mana 对话窗口）：**

| 写法 | 结果 |
|---|---|
| `![](file:///C:/...)` 本地路径 | ✅ 可用 |
| `![](https://...)` 远程直链 | ✅ 可用 |
| `<img src="..." width="140">` | ❌ **不可用**——渲染器会把标签当纯文本原样打出来 |
| 动图 `.gif` | ✅ 可用，会播放 |
| 动图 `.webp` | ✅ 可用，会播放 |

**结论：Markdown 图片语法没有任何尺寸控制能力**（`{width=100}` 是 Pandoc/Kramdown 扩展，非通用标准），HTML 又被过滤，所以**尺寸只能靠图片文件本身**。

**标准：库里所有图片宽度统一为 240px。** 低于此值不缩，高于此值用 `assets/scripts/resize.ps1` 缩。240px 在对话窗口里的观感约等于微信表情包，实测用户认可。

## 格式：动图一律用 WebP，不用 GIF

| 格式 | 用途 | 说明 |
|---|---|---|
| `.jpg` | 静图 | 240px 宽，质量 90 |
| `.webp` | **动图** | 240px 宽，q75；支持 alpha |

**动图不要存 GIF。** 实测同一批素材（2026-10-08）：

| | 体积 | 画质 |
|---|---|---|
| 原始 GIF | 3314KB | 最好 |
| GIF 压到 16 色 | 1376KB | 有可见色带，照片类素材发花 |
| **WebP q75** | **719KB** | 全彩，肉眼无差 |

GIF 只有 256 色调色板、没有帧间压缩，实拍素材会膨胀到离谱（167×167 的图能到 935KB）。WebP 全彩还小三倍，没有理由用 GIF。

## 索引位置

按顺序试，**第一个取得到的就用**：

1. **本地**（只有库主有）：`C:\Users\Admin\Documents\Stickers\index.json`
2. **远程**（所有人）：<https://raw.githubusercontent.com/thisissanpai/aimana-stickers/main/index.json>

使用者机器上只有远程，用 WebFetch 拉取即可。**取不到索引就别硬凑图**，直接不加。

## 加新图（仅库主）

1. 图片丢进 `C:\Users\Admin\Documents\Stickers\`。**文件名用英文/拼音且语义化**（如 `pig-flat.webp`）——中文名进 URL 要百分号编码，GitHub 与 Gitee 处理不一致，容易间歇性裂图（本地看着好好的，同事那边裂）。
2. **静图**跑缩图脚本（原图备份进 `_originals\`）：
   ```powershell
   & "C:\Users\Admin\.claude\skills\sticker\assets\scripts\resize.ps1"
   ```
3. **动图（GIF）**跑转换脚本——缩到 240px 并转成 WebP，原 GIF 备份后删除：
   ```powershell
   & "C:\Users\Admin\AppData\Local\.aimana\bin\python\python.exe" `
     "C:\Users\Admin\.claude\skills\sticker\assets\scripts\gif_to_webp.py"
   ```
4. **用 Read 工具实际打开图片看图**，再在 `index.json` 的 `stickers` 数组补一条：`类型` / `含义` / `情绪` / `适用` / `禁忌`。
5. 禁止凭文件名猜测含义——必须看图。
6. **动图只能看到第一帧，含义常看不全**（配字往往在后面几帧）。不确定就用 Pillow 的 `ImageSequence` 抽几帧确认，别猜。

> ⚠️ 含中文的 `.ps1` **必须存成 UTF-8 带 BOM**。PowerShell 5.1 对无 BOM 的脚本按 GBK 解码，中文会乱码并**破坏语法**——报错看着像括号写错，实际是编码问题。用 `[System.IO.File]::WriteAllText($p, $c, (New-Object System.Text.UTF8Encoding($true)))` 写。
> ⚠️ 两个 `.py` 脚本依赖 **Pillow**（已装；重装用 `python -m pip install Pillow`）。`resize.ps1` 走 System.Drawing，**碰不了动图**——只能取第一帧，会把动图变成静图，所以动图必须走 `gif_to_webp.py`。
> ⚠️ Pillow 的 `Image.open` 是惰性加载、**不关句柄就覆盖/删除同一文件会报 WinError 5**（拒绝访问，等于自己锁自己）。脚本里已用 `with` 处理。

## 当前状态

- **已上云**：<https://github.com/thisissanpai/aimana-stickers>（公开仓库，2026-10-08）
- 库现有 **9 张**（5 静图 + 4 动图），合计约 773KB。
- 已覆盖情绪：尴尬心虚、郑重道歉、交付呈上、赔笑讨好、委屈、亲昵凑近、无语呆滞、摇人、躺平。
- **缺口**：开心/点赞、震惊、催促、加油打气——碰到这些场景只能不加图。
- **加图流程**：图片丢进 `C:\Users\Admin\Documents\Stickers\` → 跑缩图/转格式脚本 → 看图补 `index.json` → `git add -A && git commit && git push`。推送后同事那边自动拿到。
- **本地仓库就是仓库根**：`C:\Users\Admin\Documents\Stickers\` 本身是 git 工作区（remote `origin` 指向上面的地址，分支 `main`）。

## 推送仓库（仅库主）

```powershell
$git = "C:\Users\Admin\.local\MinGit\cmd\git.exe"
$env:GIT_SSH_COMMAND = "C:/Windows/System32/OpenSSH/ssh.exe"   # 必须正斜杠
& $git -C "C:\Users\Admin\Documents\Stickers" add -A
& $git -C "C:\Users\Admin\Documents\Stickers" commit -m "..."
& $git -C "C:\Users\Admin\Documents\Stickers" push
```

> ⚠️ `GIT_SSH_COMMAND` **必须用正斜杠**。写反斜杠会被 git 内部 shell 吞掉，报 `C:WindowsSystem32OpenSSHssh.exe: command not found`。
> 认证走 SSH 密钥 `C:\Users\Admin\.ssh\id_ed25519`（已加到 GitHub 账号，无密码短语——卡在提示上会让非交互环境挂死）。
