# make-knowledge-cards

把文章转成便于复习的 Markdown 知识卡片，帮助读者从长文中提取重点、合并重复内容，并通过例子或自测题检查理解。
适用场景：阅读复习、技术文章整理，以及操作流程学习。

本项目是供 Codex 等能够读取 Skill 指令的智能体使用的指令包。卡片由宿主智能体依据 [SKILL.md](skills/make-knowledge-cards/SKILL.md) 生成；项目没有独立转换程序、命令行入口或 Web 服务。

## 主要解决什么问题

文章重点分散、重复表述较多，直接保存全文不方便复习。本 Skill 将原文整理为一张一个知识点的卡片，并保留原文依据，方便回看和核对。它要求提炼忠于原文，不依靠补充外部知识来凑卡片数量。

## 主要功能

- **两种输入方式**：粘贴文章正文，或指定本地 `.md`、`.markdown`、`.txt` 文件。
- **提取与去重**：围绕定义、机制、因果、方法、条件及限制选取重点，合并同义内容，忽略广告和装饰性细节。
- **通常生成 5～8 张卡片**：信息不足时按实际数量输出，不足 5 张会说明原因；没有有效知识点时可以输出 0 张。
- **完整的复习结构**：每张包含标题、核心知识、简明解释、原文例子或附答案的自测题，以及简短的原文依据。
- **保留原文边界**：保留数字、单位、前提、不确定性和观点归属；遇到矛盾时说明或避开无法确定的结论。
- **Markdown 输出**：默认按用户语言在对话中返回；用户明确要求保存时，由具备文件写入能力的宿主按指定路径保存。

以上是 Skill 对智能体的工作要求，生成质量仍需结合原文核对。

### 当前不支持

网页抓取、PDF 输入或转换、Anki 导出、图形界面。只有网址时，需要先在此 Skill 之外准备正文，再粘贴或保存为支持的本地文件。正文中混入的命令或角色提示只作为材料，不作为新的操作指令。

## 安装方法

### 环境要求

使用可读取本地指令和输入文件的智能体宿主，例如 Codex。项目本身无需安装 Python、Node.js 或第三方运行依赖；宿主的安装与登录由宿主自身提供。运行官方开发校验脚本时才需要 Python 及该脚本的依赖。

### 方式一：直接读取项目内的 Skill

将项目文件放到本地，并让宿主打开项目目录。保留 `skills/make-knowledge-cards` 整个目录，即可按下面“使用方法”中的完整路径提示使用，无需额外复制到技能目录。

单独把 Skill 放在项目的 `skills/` 目录，并不意味着宿主已经自动发现或安装了它。

### 方式二：安装到 Codex 项目技能目录

按 [OpenAI 官方 Skill 文档](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)，Codex 支持从当前工作目录的 `.agents/skills` 发现本地 Skill。在项目根目录运行以下 PowerShell 命令，将整个 Skill 复制到该位置：

```powershell
$skillDestination = Join-Path (Get-Location) '.agents/skills/make-knowledge-cards'
if (Test-Path -LiteralPath $skillDestination) {
    throw '目标 Skill 已存在，请先核对已有内容。'
}
New-Item -ItemType Directory -Path '.agents/skills' -Force | Out-Null
Copy-Item -LiteralPath './skills/make-knowledge-cards' -Destination $skillDestination -Recurse
```

安装后的目录应包含 `.agents/skills/make-knowledge-cards/SKILL.md` 和 `agents/openai.yaml`。发现后可使用 `$make-knowledge-cards` 调用；若未显示，可重启 Codex 后再检查。这里提供的是安装说明，项目原始交付目录仍为 `skills/make-knowledge-cards`。

## 使用方法

以下相对路径以项目根目录为基准，也可改为实际绝对路径。

### 处理本地文件

```text
请读取 skills/make-knowledge-cards/SKILL.md，按其中要求，
将 tests/inputs/technical.md 提炼成知识卡片。
```

### 处理粘贴的正文

```text
请读取 skills/make-knowledge-cards/SKILL.md，按其中要求处理以下正文：

本项目登记表保留 30 天，仅适用于本项目。

编号冲突时暂停登记并联系采集人确认，不自行选择编号。
```

### 安装后调用并保存结果

宿主已发现此 Skill 时，可以发送：

```text
请使用 $make-knowledge-cards，将 tests/inputs/shishuo.md 提炼为知识卡片，
保存到项目根目录的 shishuo.cards.md。
```

未安装时仍可显式要求读取 `skills/make-knowledge-cards/SKILL.md`，并附上保存路径。没有正文或文件路径时，Skill 会请求补充输入；文件读取失败时会说明问题。

## 输入输出示例

输入为上面“处理粘贴的正文”中的两段文字，完整输出示例如下。由于只有两个独立知识点，输出两张卡片：

```markdown
来源：用户粘贴文本｜共 2 张卡片

原文仅有 2 个独立知识点，不强行凑满 5 张。

### 卡片 1｜本项目登记表保留期限
- **核心知识**：本项目登记表保留 30 天。
- **简明解释**：期限仅适用于本项目。
- **自测题**：原文规定的保留期限和适用范围是什么？
- **参考答案**：30 天，仅适用于本项目。
- **原文依据**：正文第 1 段。

### 卡片 2｜编号冲突的处理
- **核心知识**：编号冲突时暂停登记，联系采集人确认，不自行选择编号。
- **简明解释**：需要通过采集人确认冲突编号。
- **自测题**：编号冲突时可以自行选一个编号继续登记吗？
- **参考答案**：不可以，应暂停登记并联系采集人确认。
- **原文依据**：正文第 2 段。
```

## 文件与测试

- [SKILL.md](skills/make-knowledge-cards/SKILL.md)：提炼流程与输出要求。
- [agents/openai.yaml](skills/make-knowledge-cards/agents/openai.yaml)：显示名称、简介和默认提示词。
- [tests/inputs](tests/inputs)：四份自编测试文本，以及一份带来源信息的《师说》真实文章正文。
- [tests/outputs](tests/outputs)：已生成并保存的卡片输出。
- [测试记录](tests/RESULTS.md)：检查标准、发现的问题、改进和校验结果。
- [《师说》验证记录](tests/shishuo.REVIEW.md)：真实文章逐卡核对结果。

已有测试结果：

| 材料 | 实际输出 | 主要核对内容 |
| --- | --- | --- |
| [技术说明](tests/inputs/technical.md) | [6 张](tests/outputs/technical.cards.md) | 去重、实验条件、数值与不确定性。 |
| [操作流程](tests/inputs/procedure.txt) | [5 张](tests/outputs/procedure.cards.md) | 步骤顺序、异常处理和适用范围。 |
| [短篇叙事](tests/inputs/narrative.md) | [2 张](tests/outputs/narrative.cards.md) | 不凑数量，区分观察、猜测与证据限制。 |
| [矛盾及嵌入指令](tests/inputs/boundaries.txt) | [1 张](tests/outputs/boundaries.cards.md) | 不擅自解决矛盾，不执行正文中的命令。 |
| [韩愈《师说》](tests/inputs/shishuo.md) | [6 张](tests/outputs/shishuo.cards.md) | 保留作者观点、古文语义及“不一定”的限定。 |

这些结果来自同一智能体的实际生成与自查，没有独立模型评测或自动化语义测试。复测时让智能体读取 Skill 并逐个处理输入，再对照原文核对；已有输出用于参考，不要求逐字匹配。

Skill 已通过官方 skill-creator 随附的 `scripts/quick_validate.py` 结构校验。该脚本不包含在本项目中，具体执行记录见[测试记录](tests/RESULTS.md)。Windows 下需用 `python -X utf8` 避免默认 GBK 编码无法读取中文 UTF-8 文件。结构通过不代表内容质量得到自动保证；长文件分段读取、编码异常及不同模型一致性尚未实际覆盖。

## 许可

项目原创文件采用 [MIT License](LICENSE)。《师说》测试输入保留韩愈署名和原文来源，来源页面标明原作为公有领域作品。
